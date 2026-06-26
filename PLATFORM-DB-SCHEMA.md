# Platform Database Schema

_Living document. Last updated 2026-06-26._

This document describes, **conceptually**, the relational (MariaDB/MySQL) schema behind the [Open Parliament TV Platform](https://github.com/OpenParliamentTV/OpenParliamentTV-Platform): what the tables are, how they group, and how they relate. It is the database counterpart to [PLATFORM-DATAOBJECTS.md](PLATFORM-DATAOBJECTS.md) (the JSON:API objects the platform serves) and [DATA-STRUCTURES.md](DATA-STRUCTURES.md) (the cross-parliament data model).

Exact column-level DDL is **not** repeated here; it lives in the canonical SQL dumps in the Platform repo (see [Canonical schema files](#canonical-schema-files)). The table names below are the defaults.

---

## 1. Two-database design

The platform splits its relational data across **two kinds of database**:

- **One platform database** (default `openparliamenttv`): the cross-parliament, shared layer, holding the linked **entities** (people, organisations, terms, documents), platform-internal bookkeeping (users, permissions, data-quality review), and cross-cutting features (notifications, API infrastructure).
- **One parliament database per parliament** (default `openparliamenttv_<code>`, e.g. `openparliamenttv_de`): the **proceedings** of a single parliament, namely its electoral periods, sessions, agenda items, media (speeches), transcripts, and timecoded annotations.

The default deployment runs **one parliament per instance** (a single platform DB and a single parliament DB), and this is the more likely setup. The architecture also allows several parliaments on the same instance, in which case that instance runs **one platform DB and N parliament DBs** (one per configured parliament). Either way, entities are shared because the same person or organisation appears across parliaments and is identified globally (by Wikidata Q-number), while media and proceedings are partitioned because they are parliament-specific.

```
              ┌────────────────────────────┐
              │  platform DB               │   shared, cross-parliament
              │  (openparliamenttv)        │
              │  entities · users · …      │
              └─────────────┬──────────────┘
                            │ linked by Wikidata Q-ID
        ┌───────────────────┼───────────────────┐
┌───────┴────────┐  ┌───────┴────────┐  ┌────────┴───────┐
│ parliament DB  │  │ parliament DB  │  │ parliament DB  │  media, proceedings,
│ …_de (Bundestag)│ │ …_at (Austria) │  │ …              │  one per parliament
└────────────────┘  └────────────────┘  └────────────────┘
```

---

## 2. Platform database

### 2.1 Entities (the public, linked data)

The four entity types that back the public entity pages and the API `getItem`/`search` responses. Each is identified by its **Wikidata Q-number** (`varchar`), except `document` which uses an internal auto-increment id and carries a separate `DocumentWikidataID`.

| Table | Represents |
|---|---|
| `person` | A speaker / politician. Carries names, abstract, thumbnail, website, and party/faction organisation references. |
| `organisation` | A party, faction, government body, etc. Carries label, type, colour, and display ordering/filter flags. |
| `term` | A topical/subject term (a controlled concept) speeches can be about. |
| `document` | A legal or official document (`DocumentType` = `legalDocument` | `officialDocument`) a speech can be based on. |

All four share a common shape: `Label`, `LabelAlternative`, `Abstract`, `Thumbnail*` (URI/creator/license), `AdditionalInformation` (JSON), and a `LastChanged` timestamp. Within the platform DB, `person` references `organisation` by id (`PersonPartyOrganisationID`, `PersonFactionOrganisationID`).

### 2.2 Platform internals

| Table | Represents |
|---|---|
| `user` | Platform accounts (admins/editors). Holds login, role, password hash + pepper, activation/blocked flags. |
| `auth` | Per-user authorization grants (action / entity scope) consulted by the auth layer. |
| `conflict` | Queued data-quality conflicts for manual review during import. |
| `entitysuggestion` | Suggested new entities awaiting curation. |

### 2.3 Notifications & alerts

A self-contained, optional feature (gated by `$config["allow"]["notifications"]`). Users save searches as **alerts** and receive **notifications** in-app or by email digest.

| Table | Represents |
|---|---|
| `alert` | A standing saved-search subscription (criteria as JSON, frequency, channels). |
| `notification` | An individual delivered message (an alert match, a broadcast, or a system event); a unique `(AlertID, MediaID)` key dedups alert matches across re-imports. |
| `system_message` | Admin-authored broadcasts / automated events that fan out into notifications. |
| `notification_preference` | Per-user global preferences (one row per user), including the email-unsubscribe token. Kept separate from `user` so the unauthenticated unsubscribe flow never touches the auth record. |

### 2.4 API infrastructure

| Table | Represents |
|---|---|
| `apiratelimit` | Fixed-window request counter for the public API, one row per client key (a salted hash of the caller's IP, or `key:<ApiKeyID>` when an API key is used). |
| `apikey` | Public API keys. A key grants a **raised rate limit** on public read endpoints (it is not an authentication mechanism). Only a hash of the secret is stored. |

---

## 3. Parliament database

A single parliament's proceedings, modelled as a temporal hierarchy that bottoms out at an individual speech and its transcript:

```
electoralperiod          ElectoralPeriodID (e.g. "DE-20")
  └── session            SessionElectoralPeriodID → electoralperiod
        └── agendaitem   AgendaItemSessionID → session
              └── media  MediaAgendaItemID → agendaitem      (one media = one speech)
                    ├── text         TextMediaID → media     (the transcript body)
                    └── annotation   AnnotationMediaID → media (timecoded links & markers)
```

| Table | Represents |
|---|---|
| `electoralperiod` | A legislative term (`ElectoralPeriodNumber`, start/end dates). |
| `session` | A sitting within an electoral period. |
| `agendaitem` | An agenda item within a session (`OfficialTitle` + display `Title`). |
| `media` | A single **speech**: its video/audio URIs, duration, thumbnail, public flag (`MediaPublic`), and alignment status. The central unit the search index and player are built around. |
| `text` | The **transcript** of a media item (`TextBody` as `mediumtext`, plus language, license, hash). |
| `annotation` | Timecoded annotations on a media item, including the **entity links** (speaker, mentioned people/organisations/terms/documents) that connect a speech back to the platform entities. |

This mirrors the generic five-level model described in [DATA-STRUCTURES.md](DATA-STRUCTURES.md); see that document for where individual parliaments diverge from it.

---

## 4. How the two databases connect

There is **no foreign key across databases**. The bridge is the **Wikidata Q-number**: an `annotation` row in a parliament DB references a platform entity by id via `AnnotationResourceID` (e.g. a person's `Q`-number), and the platform resolves it against `person` / `organisation` / `term` / `document` in the platform DB. The same id model lets one entity be shared across every parliament instance.

So a fully-resolved speech object (see [PLATFORM-DATAOBJECTS.md](PLATFORM-DATAOBJECTS.md)) is assembled by joining **parliament-DB** rows (`media` → `text` → `annotation` → `agendaitem`/`session`/`electoralperiod`) with **platform-DB** entities looked up by the Q-numbers carried in the annotations.

Full-text search and aggregated statistics are served from **OpenSearch**, not these tables; the relational schema is the system of record that the OpenSearch indices are built from.

---

## Canonical schema files

The authoritative, column-level DDL lives with the code it is coupled to, in the Platform repo:

- **Platform DB**: [`db/openparliamenttv_platform.sql`](https://github.com/OpenParliamentTV/OpenParliamentTV-Platform/blob/main/db/openparliamenttv_platform.sql)
- **Parliament DB**: [`db/openparliamenttv_parliament.sql`](https://github.com/OpenParliamentTV/OpenParliamentTV-Platform/blob/main/db/openparliamenttv_parliament.sql)

These are fresh-install dumps; import the platform dump once into the platform database and the parliament dump once per parliament database. This document is conceptual and should be updated when the canonical dumps change.

## Related documents

- [PLATFORM-DATAOBJECTS.md](PLATFORM-DATAOBJECTS.md): the JSON:API objects assembled from these tables.
- [PLATFORM-API.md](PLATFORM-API.md): the REST API that serves them.
- [PLATFORM-URLS.md](PLATFORM-URLS.md): the public URL surface.
- [DATA-STRUCTURES.md](DATA-STRUCTURES.md): the cross-parliament data model and its divergences.
