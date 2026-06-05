# Cross-Parliament Data Structures

_Living document. Last updated 2026-06-05._

This document surveys the **structural divergences** across the parliaments Open Parliament TV has either implemented or evaluated as integration candidates. It covers the places where the generic hierarchy or ID model leaks, where one parliament's source shape doesn't fit the others, and where the platform's assumptions break down. It is background for [STAGE2-FORMAT.md](STAGE2-FORMAT.md), which defines the Stage 2 JSON format itself; field-level schema details (exact ID strings, faction shapes, language codes) belong there, not here.

The focus is the **shape** of each parliament's data (temporal hierarchy, ID model, media↔speech relationship, agenda granularity), not the scraping mechanics that produce it.

---

## 1. The generic 5-level model

The platform and pipeline share an implicit hierarchy:

```
Parliament
  └── Electoral Period
        └── Session
              └── Agenda Item
                    └── Speech (with Speakers, Media, Text, Documents)
```

The platform encodes the first four levels into a single composite URL ID:

```
DE-0180231031
└┬─┘└┬┘└─┬─┘└┬┘
 │   │   │   └─ media seq (3 digits, per-session auto-increment at import time)
 │   │   └───── session number (4 digits, zero-padded)
 │   └───────── electoral period (3 digits, zero-padded)
 └───────────── parliament code (1+ hyphenated segments, e.g. DE, DE-RP)
```

The detail endpoint parses this purely by length (`getInfosFromStringID()` in `modules/utilities/functions.php`): length ≥ 8 = media, 7 = session, 3 = period. No regex enforces the slot widths.

There is no `parliament` table; parliaments live in `config.php` under `$config["parliament"][CODE]`.

This model **assumes the Bundestag shape**: a small-integer sequential Wahlperiode, a small-integer Sitzung within it, one MP4 per speech, one language per text, German title vocabulary for search filters. None of the other parliaments fit cleanly.

---

## 2. Per-parliament structural summary

For each parliament: the structural facts that differ from the Bundestag baseline. Parliaments are grouped as **supranational**, **national**, and **regional** (the German Landtage), which are clustered together at the end.

### Supranational

#### 2.1 EU (European Parliament): multiple structural mismatches

1. **"Session" is a plenary day, not a sitting.** EU plenaries run 3 sittings (morning / afternoon / evening) under one daily agenda. The pipeline collapses this into one session per day, losing the sitting split. `session.number = int("YYYYMMDD")` (e.g. `20251008`) is an 8-digit value that **does not fit the platform's 4-digit session slot**.
2. **Media is not 1:1 with speech.** Every speech in a sitting shares the same HLS master playlist (`videoFileURI`) and is differentiated only by `media.additionalInformation.startOffset` (seconds). The platform's `media` table has no offset column; the UI cannot seek to it.
3. **Multilingual by nature.** Each speech is delivered in one of 24 languages with simultaneous interpretation, but the pipeline emits a single `textContents[]` (English) and hardcodes `originalLanguage: "en"`. The source exposes all 24 language renderings; none are currently captured.
4. **Term hardcoded.** The electoral period is a constant in the merger; a new term requires a code change.
5. **Rich agenda vocabulary.** EU emits `agendaItem.number` (integer 1..N per day). Useful structurally; nothing else currently uses it.

### National parliaments

#### 2.2 DE (Deutscher Bundestag, reference)

- The model fits because the model was built for it.
- One Sitzung per day, one MP4 per speech, sequential Wahlperiode (small integer), German.

#### 2.3 ES (Congreso de los Diputados)

- Sequential legislature. Per-session structure matches DE.
- One subtle structural quirk: the **proceedings-side never carries per-turn agenda titles** (everything is "Sesión plenaria"); only the media-side classification supplies meaningful `agendaItem.title` / `type`. So the agenda granularity comes from a different source than DE.
- Coarse timing (HH:MM only) limits alignment quality.

#### 2.4 SE (Sveriges Riksdag): electoralPeriod semantic mismatch

1. **`electoralPeriod.number = 2025`** is the *riksmöte* start year, not a sequential term. Other parliaments use small integers. The 4-digit year **does not fit the platform's 3-digit period slot**. SE actually has two layers of "period": the *valperiod* (4-year electoral term) and the *riksmöte* (annual session within it, Sept–Sept). The model has no way to express this.
2. **`agendaItem.id` is a cross-session "matter" reference.** SE emits `agendaItem.id: "HD01UbU20"`, a Riksdag matter ID (a legislative topic that can span several sessions), giving agenda items a cross-session identity the model otherwise lacks.
3. **Media URI carries `#t=start,end` Media Fragment.** SE addresses per-speech windows by URI fragment on a per-debate MP4. Same structural issue as EU (media not 1:1 with speech), solved at the URI level instead of via metadata.
4. **`isReply` boolean** distinguishes brief follow-up turns within a debate. Captures structure that DE/EU/ES would also have but don't model.
5. **`originPersonID`** carries the Riksdag intressent_id, a parliament-native person identifier emitted alongside the resolved entity ID.

#### 2.5 NO (Stortinget): three-level term hierarchy, globally-unique meeting IDs

1. **Three-level temporal hierarchy.** Stortinget natively has `stortingsperiode (4-year term) > sesjon (1-year session, Oct–Sept) > møte (single meeting)`. The Stage 2 model has two levels (electoralPeriod + session); the sesjon level is lost. Same structural problem as SE: two layers of "period", model has one. See §3.4 / §4.2.
2. **Stortingsperiode is a year-range string** (`"2025-2029"`, …), not an integer. The implementation maintains an internal lookup mapping it to a synthetic term integer. The §3.4 redesign would let us emit `{"number": 22, "label": "2025-2029", "type": "term"}` directly.
3. **`moteid` is globally unique, not session-scoped.** Stortinget assigns each meeting a 5-digit ID (`11518`) unique across all sesjoner and stortingsperioder: the natural primary key, also what the video archive keys on. It does not fit the platform's 4-digit `session.number` slot. The implementation falls back to an ordinal within the sesjon-year, which fits the slot but **collides across sesjon-years** within the same term. Resolution requires either §3.4's electoralPeriod redesign (sesjon as `subPeriod`, scoping `session.number` to it) or encoding `sesjon_index * 1000 + ordinal` as TW does.
4. **`personID` is a short alphanumeric code** (`"JGS"`, `"MARNIL"`, `"EW"`), persistent across sesjoner and stortingsperioder, emitted as `people[].originPersonID`. Reinforces the case in §4.4 for promoting `originPersonID` to a first-class field.

#### 2.6 TW (Legislative Yuan, 立法院): three-level session decomposition

1. **3-level meeting hierarchy** packed into `session.number`. TW's natural structure is `term (屆) > session period (會期) > meeting (會次)`: three levels where DE has one ("Sitzung"). The pipeline encodes this as `session_period * 1000 + meeting_number` → e.g. `5011`. The decomposition survives in `meta.meetingCode: "院會-11-5-11"` (TW-only field) but is information-lossy in the platform.
2. **No per-speech agenda.** All speeches in a session share the same `agendaItem`; the meeting title is repeated on every item. The platform's per-session agenda model assumes more granularity than the source provides.
3. **No faction concept emitted.** TW members have party affiliations in the source records, but they're not surfaced. Whether this reflects a real difference in how Taiwanese parliamentary speech is organised (no parliamentary "groups" in the EU sense) or just an unimplemented step needs domain confirmation.
4. **Whisper transcripts, not stenographic.** Text comes from automatic transcription (already aligned), not from an official Hansard. Affects text fidelity expectations.

#### 2.7 FI (Eduskunta): three-source join, term-vs-session-year period mismatch

- **Three-source merge.** Where every other parliament joins two streams (proceedings + media), FI joins **three**: (1) the broadcast `speakers[]` array (per-speech video offsets + speaker + party + agenda + reply flag) as the **media spine**, (2) the plenary-minutes XML for verbatim text, and (3) an external person register for entity resolution. The first two are joined on `personNumber` plus speech start time (`#t=`-fragment shape as SE/EU/NO); the third stream is the addition.
- **Two layers of "period", model has one (same class as SE).** Sessions are keyed by *valtiopäivät* (parliamentary year) + a per-year number that resets each year (`2026/58`), while the *vaalikausi* (4-year term) spans several years. The model flattens to one `electoralPeriod.number` = term start year (`2023`) and encodes `session.number = (year - 2023) * 1000 + number` so the per-year numbering doesn't collide across the term (TW/NO composite-encoding class). The valtiopäivät year is the dropped intra-term level. `electoralPeriod.number = 2023` is a 4-digit value that **does not fit the platform's 3-digit period slot**, the identical slot-width problem to SE (§3.1).
- **Bilingual, handled as metadata only.** The minutes mark per-speech language (fi/sv). Stage 2 records `originalLanguage` per speech, but the pipeline runs uniformly in Finnish; Swedish-minority speeches degrade slightly. Reinforces §4.6.

#### 2.8 FR (Assemblée nationale): single-source spine

- **Single-source spine.** Every other parliament joins ≥2 streams; FR's text **and** per-speech video offsets come from the *same* document: the compte rendu carries verbatim text, the agenda tree, the speakers, *and* the per-speech offset (seconds into the séance recording). The merge is therefore a single ordered walk with no cross-source matching (contrast DE's Needleman-Wunsch, EU/FI's timestamp join). The only external lookup is the séance's one video URL.
- **The per-speech offset is populated only after the recording is synced** (~next day). A just-held séance's compte rendu has no offset, so the merger falls back to the séance start (all speeches share `dateStart`, `startOffset = 0`) until a later re-run picks up the offsets. This is a *temporal* data-availability gap within a single source, distinct from the structural gaps elsewhere, and a case where re-running the same stage on the same session legitimately enriches it as upstream data matures.
- **Two layers of "period", but the term stays a clean integer (unlike SE/FI).** Sessions sit under an annual *session ordinaire/extraordinaire* whose séance number resets each year, nested in the 5-year legislature. FR keeps `electoralPeriod.number = 17` (sequential term, fits the slot) and pushes the per-year uniqueness into `session.number = (year - 2024) * 1000 + type_offset + num` (composite-encoding class of TW/NO/FI). So FR has the same dropped intra-term level as SE/FI/NO but, unlike SE and FI, does **not** pick a year for `electoralPeriod.number`.
- **One Stage 2 session = one séance, not one calendar day.** The AN runs several séances per day (1ère/2ème/3ème séance), each with its own compte rendu and its own video, so the séance (not the day) is the natural session unit.
- **Media is not 1:1 with speech** (EU/SE/FI/NO class): one HLS master per séance, per-speech window via `#t=start,end` fragment + `additionalInformation.startOffset`. Same §3.2 issue, solved the SE way (offset in the URI).

#### 2.9 PT (Assembleia da República): media-spine + verbatim-text two-source merge

- **Two-source merge with a *media* spine and a separate verbatim-text stream: the DE join shape, but the spine is the media side.** Unlike DE (proceedings spine ⋈ media) PT joins (1) the video API as the spine (one *intervention* per speech with speaker + party + `interventionType` + per-speech offsets) against (2) the verbatim debate text. The join is a Needleman-Wunsch alignment on a per-turn key because the text is finer-grained than the video list (it interleaves chair interjections the video list does not enumerate). Same NW class as DE/ES; the difference is that the media stream, not the text, defines the speech set, while the text is grafted on.
- **Per-speech video is a server-side-clipped HLS stream** per speech, built from the JSON start/end times. So PT is "media not 1:1 with the *session* recording" (§3.2) but the player gets a real per-speech clip URL with no offset columns or fragment needed. The un-clipped session recording is the alignment audio source (FR/EU pattern).
- **`electoralPeriod.number = 17` stays a clean sequential integer (FR class), not a year.** Two layers of "period" (legislatura + *sessão legislativa*, the parliamentary year, where reunião numbers reset each year), but, like FR, the term stays integer and the dropped intra-term level is folded into `session.number = sessão_legislativa * 1000 + reunião` (TW/FI/FR composite-encoding class).
- **One Stage 2 session = one reunião plenária** (a calendar-day sitting).

### Regional parliaments (German Landtage)

#### 2.10 DE-RP (Landtag Rheinland-Pfalz)

- ~5-digit `meta.session` (`"18085"`) and same shape as DE, with no structural divergence at the hierarchy level. The resulting Stage 2 conforms to the Bundestag-shaped model.

#### 2.11 DE-ST (Landtag Sachsen-Anhalt): three-level temporal hierarchy (Sitzungsperiode dropped)

- **Three-level temporal hierarchy.** The native model is `Wahlperiode > Sitzungsperiode > Landtagssitzung > TOP > Redebeitrag`. Sitzungsperiode is a 1–3 day plenary block grouped under a single portal URL; Landtagssitzung is the calendar-day-level unit canonical for Plenarprotokoll citations. The Stage 2 model flattens to `electoralPeriod > session`, so Sitzungsperiode is recorded in `meta.sitzungsperiode` / `debug.*` only and not first-class, the same intra-term-grouping pattern as NO (`sesjon`) and SE (`riksmöte`). 5-digit `meta.session` (`"08105"`) keyed by Landtagssitzung; matches DE/DE-RP shape at the schema level.
- **Canonical session number is not exposed as structured data.** The portal renders Sitzung numbers only in transcript prose ("Hiermit eröffne ich die NNN. Sitzung") and Tagesordnung / Plenarprotokoll PDF headers, never as a `data-*` attribute or JSON field. The pipeline derives it structurally from a cumulative day-count across the archive.

#### 2.12 DE-SH (Landtag Schleswig-Holstein): three-level temporal hierarchy, video-only

- **Three-level temporal hierarchy.** The native model is `Wahlperiode > Tagung > Landtagssitzung > TOP > Redebeitrag`. Tagung is a 1–3 day plenary block; Landtagssitzung is the calendar-day-level unit canonical for Plenarprotokoll citations. The Stage 2 model flattens to `electoralPeriod > session`, so Tagung is recorded in `meta.tagung` / `debug.*` only and not first-class, the same intra-term-grouping pattern as DE-ST, NO (`sesjon`), SE (`riksmöte`). 5-digit `meta.session` (`"20119"`) keyed by Landtagssitzung; matches DE/DE-RP/DE-ST shape at the schema level.
- **Video-only.** Plenarprotokolle are PDF-only and there is no PDF parser yet, so DE-SH ships with `textContents: []`: the media stream carries per-speech identity, the text stream does not. This is structurally distinct from DE-BE: DE-SH has a per-speech video spine that lets us ship the video half today, while DE-BE has neither text nor per-speech video. See the three-regime framing in §4.7.
- **Media not 1:1 with speech.** Per-speech windows on a shared recording are addressed with an HTML5 media-fragment URI (`#t=start,end`), the same `#t=` solution SE uses, but present in the source data rather than synthesised by the merger.
- **Agenda granularity.** A per-speech `(TOP-no, thema)` pair; the merger emits one `agendaItem.id = "TOP-{n}"` per speech, so the agenda is effectively per-speech, slightly over-resolved (§3.5).

#### 2.13 DE-BE (Abgeordnetenhaus von Berlin): no source-side per-speech spine

- **The structural case where no source format provides per-speech segmentation.** The proceedings are published as Plenarprotokoll PDFs only; the open-data XML is metadata + page-anchored agenda only, with zero proceedings text.
- **Video: session-level only.** The available recordings split each session into a few coarse segments (or post the full session), with no per-speech granularity on either channel.
- DE-BE is the most extreme case of the structural problem in §3.2 (media not 1:1 with speech): both media *and* text are session-level. The EU/SE startOffset workaround does not apply without first deriving a spine. See the three-regime framing in §4.7.

#### 2.14 DE-BY (Bayerischer Landtag): video-only, native per-speech HLS masters

- **Video-only (DE-SH regime):** per-speech video spine + PDF-only proceedings → ships `textContents: []`. The §4.7 "one stream has per-speech identity, the other doesn't" case (video ✓, text ✗). Distinct from DE-BE (neither stream).
- **The source's *native* granularity is the speech.** DE-BY serves **one complete native HLS master playlist per speech**, with no fragment, no offset column, no clip-param. So `videoFileURI` is a directly-playable per-speech clip and the platform needs nothing new (like DE's one-MP4-per-speech but HLS). There is **no per-speech end/duration field** in the source.
- **Two-level period axis.** `electoralPeriod.number = 19` is a sequential Wahlperiode (DE class) and `session.number` = Sitzungsnr → `19{NNN}` fits the platform slots. Bavaria is a **pure two-level hierarchy** (Wahlperiode > Sitzung): unlike DE-ST (`Sitzungsperiode`) and DE-SH (`Tagung`) there is **no dropped intra-term super-level**, so no `meta.tagung`/ `subPeriod` is needed (§3.4/§4.2 do not apply).
- **Genuine per-TOP agenda granularity.** Each playlist is one Tagesordnungspunkt with its own title and its own set of speeches, so the agenda model fits naturally (one `agendaItem` per TOP), without DE-SH's per-speech over-resolution (§3.5) or TW's one-item-per-meeting coarseness.

#### 2.15 DE-BW (Landtag von Baden-Württemberg): video-only, per-part MP4 + offset model

- **Video-only (DE-SH/DE-BY regime):** per-speech video spine + PDF-only proceedings → ships `textContents: []`. The §4.7 case (video ✓, text ✗).
- **Per-speech video is the SE/DE-SH `#t=start,end` offset model:** one MP4 per Sitzung-part, with per-speech windows addressed by an HTML5 media-fragment `#t=start,end` on `videoFileURI` + `additionalInformation.startOffset/endOffset`. The per-speech **end** is synthesised as the next speech's start *within the same part* (the source carries only a start offset). Times are **video-relative** (no per-speech wall-clock in the source), so `debug.timesAreVideoRelative = true`.
- **Multi-part = a same-day Sitzung split into sequential video files** (not multi-day): a long sitting day is broken into sequential MP4s, each with its own chapter list (offsets reset per part) and **continuous** TOP numbering (a debate can straddle the break: `TOP 4` in part 1, `Fortsetzung TOP 4` in part 2). Because each part is an independently-addressable MP4, there is **no cross-file offset arithmetic**; the merger groups parts by (Sitzung, date), orders speeches by `(part, offset)`, and derives `agendaItem.id` from the parsed TOP number (so `TOP 4` / `Fortsetzung TOP 4` collapse to one `agendaItem`). Distinct from EU's intra-day sittings (§2.1, collapsed and lost): DE-BW keeps every part's speeches under the one calendar-day session, losing nothing.
- **Two-level period axis (DE-BY class).** `electoralPeriod.number = 17` is a sequential Wahlperiode and `session.number` = Sitzungsnr → `17{NNN}` fits the platform slots. Pure two-level hierarchy, with no dropped intra-term super-level (§3.4/§4.2 do not apply).
- **Genuine per-TOP agenda granularity.** Each chapter is one Tagesordnungspunkt with its own title/description and speeches (one `agendaItem` per TOP, `id = "TOP-{n}"`), without DE-SH's per-speech over-resolution (§3.5) or TW's one-item-per-meeting coarseness.

#### 2.16 DE-HH (Hamburgische Bürgerschaft): video-only, per-TOP clip, real wall-clock, source-native UUID `originID`

- **Video-only (DE-SH/DE-BY/DE-BW regime):** per-speech video spine + PDF-only proceedings → ships `textContents: []`. The §4.7 case (video ✓, text ✗).
- **Per-speech video is the SE/DE-SH `#t=start,end` offset model, but per-TOP not per-part** (cf. DE-BW): each Tagesordnungspunkt is one server-side-clipped HLS master, and per-speech windows are addressed by `#t=start,end` on that clip + `additionalInformation.startOffset/endOffset` (offsets relative to the TOP clip). A sign-language stream variant is exposed alongside the clean stream. Structurally between PT's per-speech HLS clip and DE-BW's per-part MP4 + fragment: per-TOP clip + per-speech fragment.
- **Real per-speech wall-clock.** Unlike DE-SH/DE-BY/DE-BW (video-relative offsets), each speech carries absolute wall-clock timestamps; the pipeline emits real UTC `dateStart`/`dateEnd` and sets `debug.timesAreVideoRelative = false`. The clip-relative offsets (for the media fragment) and the wall-clock (for temporal display) are two distinct reference frames, each used for its purpose.
- **Source-native per-speech identity.** A stable UUID is used directly as Stage 2 `originID` (DE-SH/DE-BY/DE-BW synthesise the id from position/offset). The speaker-function field is overloaded (the faction for MPs but a government role for senators), so the parser routes party values to `faction` and everything else to `role`.
- **Two-level period axis (DE-BY/DE-BW class).** `electoralPeriod.number = 23` is a sequential Wahlperiode and `session.number` = Sitzungsnr → `23{NNN}` fits the platform slots. Pure two-level hierarchy, with no dropped intra-term super-level (§3.4/§4.2 do not apply).
- **Genuine per-TOP agenda granularity.** Each video is one Tagesordnungspunkt with its own title and TOP number, and its speeches are joined to it; one `agendaItem` per TOP (`id = "TOP-{n}"` when numbered, else a capped title slug), without DE-SH's per-speech over-resolution (§3.5).

#### 2.17 DE-NI (Niedersächsischer Landtag): video-only, typed JSON API with per-speech stable speaker IDs and time-aligned VTT text

- **Video-only v1 (DE-SH/DE-BY/DE-BW/DE-HH output regime):** per-speech video spine → ships `textContents: []`, but **only by choice, not by source limitation**. Unlike the PDF-locked Landtage, DE-NI's source exposes machine-readable, **time-aligned** text (WebVTT subtitles per subject). Wiring VTT would re-add `textContents` *and* sentence timings without a separate alignment step, so DE-NI can leave the §4.7 regime cheaply (the merger already records `media.additionalInformation.subtitleVttURI`).
- **Per-speech video is the PT model (a per-speech server-side HLS clip), but parameterised** rather than a distinct file: a clip endpoint returns a server-side-clipped playlist of one per-Sitzung stream, so the clip URL **is** the speech, and (unlike DE-HH's per-TOP clip + `#t=` fragment, or DE-BW's per-part MP4 + fragment) there is no media fragment.
- **Real per-speech wall-clock** (DE-HH class): `dateStart`/`dateEnd` come from the stream start time + the per-speech offset, so `debug.timesAreVideoRelative = false`.
- **Source-native person identity.** Each speech carries a UUID (used as Stage 2 `originID`, like DE-HH) **and** a stable Abgeordneten-ID (used as `originPersonID`): a parliament-supplied *person* identifier, a candidate to populate the §4.4 `originPersonID` field.
- **Three-level temporal hierarchy** (the DE-ST / DE-SH class, not DE-HH's pure two-level): **Wahlperiode > Tagungsabschnitt > Sitzung**. OPTV's `session.number` is the Sitzung; the Sitzung number is unique within the Wahlperiode, so the flat `19{NNN}` key holds and the dropped intra-term super-level (Tagungsabschnitt) is carried in `meta.tagungsabschnitt`/`debug` (§3.4/§4.2 apply).
- **`fraktion` is always the party** (even the presiding chair retains their party), the opposite of DE-HH's overloaded function field. The chair/role signal lives in a separate `speechType`, which the parser maps to speaker `context`. Government members have an **empty** `fraktion` (they speak as government), so those speeches carry no faction.

#### 2.18 DE-NW (Landtag Nordrhein-Westfalen): video-only, parliament-native MdL id + real wall-clock

- **Video-only (DE-SH/DE-BY/DE-BW/DE-HH/DE-NI regime):** per-speech video spine + PDF-only proceedings → ships `textContents: []`. The §4.7 case (video ✓, text ✗).
- **Per-speech video is the SE/DE-SH `#t=start,end` offset model** (one HLS stream per session, per-speech windows by media fragment + `additionalInformation.startOffset/endOffset`), but with an **offset-source variant**: the precise start offset is *not* in the session page; it is rendered server-side only when a single speech is selected, so the precise per-speech offset is gated behind a per-speech request rather than present in a bulk feed. The rendered end is unreliable, so the merger synthesises each window's end from the next speech's start (the DE-BW approach).
- **Real per-speech wall-clock** (DE-HH / DE-NI class, not DE-SH/DE-BY/DE-BW video-relative): the session header carries the session start as a full ISO datetime, so `dateStart`/`dateEnd` = session start + offset and `debug.timesAreVideoRelative = false`.
- **Parliament-native person id** (the DE-NI / NO `personID` class): the source carries an MdL id for sitting members and a function id for chair/government speakers, emitted as `people[].originPersonID` (the §4.4 candidate field).
- **Separate `fraktion` + `funktion` fields, cleaner than DE-HH's overloaded function field:** `fraktion` is always the party (empty for chair/government), `funktion` the chair/government role. The parser routes them straight to `faction` vs `role` with no disambiguation heuristic.
- **Two-level period axis (DE-BY/DE-BW/DE-HH class).** `electoralPeriod.number = 18` is a sequential Wahlperiode and `session.number` = Sitzungsnr → `18{NNN}` fits the platform slots. Pure two-level hierarchy, with no dropped intra-term super-level (§3.4/§4.2 do not apply).

#### 2.19 DE-SN (Sächsischer Landtag): video-only, daily-stream `#t=` offset model with both offsets inline + real wall-clock

- **Video-only (DE-SH/DE-BY/DE-BW/DE-HH/DE-NI/DE-NW regime):** per-speech video spine + PDF-only proceedings → ships `textContents: []`. The §4.7 case (video ✓, text ✗).
- **Per-speech video is the SE/DE-SH `#t=start,end` offset model on a per-DAY HLS stream.** One daily HLS recording per calendar date; per-speech windows by media fragment + `additionalInformation.startOffset/endOffset`. Unlike DE-BW/DE-NW (which carry only a start offset and synthesise the end as the next speech's start), DE-SN's source gives **both** start and end offsets inline, so no end-synthesis is needed.
- **Real per-speech wall-clock** (DE-HH / DE-NI / DE-NW class, not DE-SH/DE-BY/DE-BW video-relative): each speech carries a wall-clock time, so `dateStart`/`dateEnd` are absolute (`debug.timesAreVideoRelative = false`). The clip offsets and the wall-clock are two distinct reference frames (the daily stream starts a few minutes before the first speech).
- **Two-level period axis (DE-BY/DE-BW/DE-HH/DE-NW class).** `electoralPeriod.number = 8` is a sequential Wahlperiode and `session.number` = Sitzungsnr → key `08{NNN}` fits the platform slots. Pure two-level hierarchy, with no dropped intra-term super-level (§3.4/§4.2 do not apply). (Multi-day Sitzungen exist, each its own daily HLS stream, but each speech carries its own date, so the calendar-day split is handled without a Tagung level.)
- **Per-TOP agenda from a per-speech `thema`.** Each speech carries a TOP number plus a `thema` text that is frequently the full agenda topic, so `agendaItem.title` is richer than DE-SH's thin `thema` (§3.5); `id = "TOP-{n}"`.

---

## 3. Structural inconsistencies the model can't currently express

Each finding labels the layer where the fix belongs (Tools / Schema / Platform).

### 3.1 [CRITICAL] Platform composite ID slot widths break for EU and SE

**Symptom:** Platform synthesises `SessionID = parliament + period(%03d) + session(%04d)` at import time, and `getInfosFromStringID()` parses by length.

| Parliament | period digits | session digits | composite | parser verdict |
|---|---|---|---|---|
| DE | 3 (018) | 4 (0231) | `DE-0180231` (7) | session ✓ |
| DE-RP | 3 (018) | 4 (0085) | `DE-RP-0180085` (7) | session ✓ |
| DE-ST | 3 (008) | 4 (0105) | `DE-ST-0080105` (7) | session ✓ (Sitzungsperiode lost in flattening) |
| DE-SH | 3 (020) | 4 (0119) | `DE-SH-0200119` (7) | session ✓ (Tagung lost in flattening) |
| DE-BY | 3 (019) | 4 (0054) | `DE-BY-0190054` (7) | session ✓, pure 2-level; nothing lost in flattening |
| DE-BW | 3 (017) | 4 (0118) | `DE-BW-0170118` (7) | session ✓, pure 2-level; nothing lost in flattening |
| DE-HH | 3 (023) | 4 (0018) | `DE-HH-0230018` (7) | session ✓, pure 2-level; nothing lost in flattening |
| DE-NI | 3 (019) | 4 (0080) | `DE-NI-0190080` (7) | session ✓ (Tagungsabschnitt dropped to `meta.tagungsabschnitt`) |
| DE-NW | 3 (018) | 4 (0117) | `DE-NW-0180117` (7) | session ✓, pure 2-level; nothing lost in flattening |
| DE-SN | 3 (008) | 4 (0025) | `DE-SN-0080025` (7) | session ✓, pure 2-level; nothing lost in flattening |
| ES | 3 (015) | 4 (0094) | `ES-0150094` (7) | session ✓ |
| TW | 3 (011) | 4 (5011) | `TW-0115011` (7) | session ✓ (digits fit, but `session.number=5011` is a composite encoding of session period × 1000 + meeting number) |
| EU | 3 (010) | 8 (20251008) | `EU-01020251008` (11) | **mis-parsed as media** (length ≥ 8) |
| SE | 4 (2025) | 3 (122) | `SE-2025122` (7) | length matches session, BUT period substring is split at pos 3 → period=`"202"`, session=`"5122"` → **wrong values** |
| FI | 4 (2023) | up to 4 (`3058`) | `FI-20233058` (8/11) | **same class as SE**: `electoralPeriod.number=2023` (vaalikausi start year) overflows the 3-digit period slot; `session.number` is the term-unique encoding `(year-2023)*1000+number` |
| NO | 3 (022) | up to 4 | `NO-0220001` (7) | session ✓ (digits fit) **but `session.number` collides across sesjon-years**: 25/26 møte 1 and 26/27 møte 1 both produce `NO-0220001`. Stortinget's globally-unique `moteid` (5 digits) doesn't fit the slot |
| FR | 3 (017) | 4 (`2254`) | `FR-0172254` (7) | session ✓ (digits fit); `session.number = (year-2024)*1000 + type_offset + num` (TW/FI composite-encoding class), but unlike SE/FI `electoralPeriod.number = 17` stays a clean sequential legislature |
| PT | 3 (017) | 4 (`1059`) | `PT-0171059` (7) | session ✓ (digits fit); `session.number = sessão_legislativa*1000 + reunião` (TW/FI/FR composite-encoding class); like FR `electoralPeriod.number = 17` stays a clean sequential legislatura |

*Note on DE-SH:* fits the slot cleanly: `session.number = 119` (Landtagssitzung), `electoralPeriod.number = 20`. The Tagung level (1–42 within the term) is the divergence and is dropped to `meta.tagung`, same as DE-ST's `meta.sitzungsperiode`. The §3.4 / §4.2 redesign would promote it to `electoralPeriod.subPeriod.number`.

**Fix:**
- **Tools, recommended:** Stop encoding semantic information into oversized integer fields. Allocate sequential `session.number` for EU (plenary-day counter within term). For SE, switch `electoralPeriod.number` semantics to *valperiod* (4-year electoral term) and use riksmöte as a `subPeriod` (see §4.2). For NO, scope `session.number` to the sesjon-year via `subPeriod` and use the *globally-unique `moteid`* as `originID` (avoiding the cross-sesjon collision while still fitting the slot). All are small integers that fit the slots.
- **Platform, alternative:** Replace length-based parsing with a per-parliament regex registry in config (`$config["parliament"]["DE"]["idFormat"]`). Each parliament declares its own widths. Backwards compatible for DE. Doesn't fix the data-shape mismatch (SE's year-as-period is still semantically wrong, NO's `moteid` collisions still happen at the data-model level), just stops it from crashing.

### 3.2 [HIGH] Media is not 1:1 with speech

The Bundestag baseline assumes one video per speech. Most other parliaments have one recording per session (or per debate / per meeting-part) and address per-speech windows by some offset mechanism. The platform's `media` table assumes one video per speech and has no offset columns. Four representations of the same underlying fact exist across parliaments:

- **URI media fragment** (`#t=start,end` on a shared recording): SE, DE-SH, DE-BW, DE-HH, DE-NW, DE-SN, FR. Works end-to-end today because the fragment lives in the URI and the player honours it.
- **Metadata offset** (`additionalInformation.startOffset`, no offset column on the platform side): EU. The UI cannot seek to it.
- **Server-side clip** (the source serves a per-speech clipped HLS stream): PT (per-speech clip), DE-HH (per-TOP clip + fragment), DE-NI (parameterised per-speech clip). The `videoFileURI` is a genuine per-speech URL the player streams directly, with no fragment or offset column required.
- **Native per-speech file** (the source's native granularity *is* the speech): DE-BY serves one HLS master per speech, so media *is* 1:1, and the platform needs nothing new, exactly as for DE/DE-RP.

**DE-BE is the most extreme variant:** both media *and* text are session-level, so the offset workaround cannot apply without first synthesising a spine. Reinforces the §4.5/§4.7 architectural questions.

**Fix (Schema + Platform):** Promote `media.startOffset` and `media.endOffset` (seconds, floats) to first-class Stage 2 fields. Tools-side: emit them for EU (currently buried in `additionalInformation`) and SE (currently in URI fragment). Platform-side: add `MediaStartOffset` / `MediaEndOffset` columns; player honours them via HLS `#t=` fragments. The server-side-clip and native-per-speech-file parliaments need nothing new.

### 3.3 [HIGH] EU originalLanguage hardcoded; multilingual `textContents` never emitted

EU emits a single English `textContents[]` with `originalLanguage: "en"` regardless of the actual speech language. The source exposes verbatim text in all 24 EP languages, none captured. The platform's `text` table is already keyed on `(MediaID, TextType, TextLanguage)` and supports multiple rows per media, so the infrastructure is there.

**Fix (Tools):** Emit the real per-speech `originalLanguage`, and one `textContents[]` entry per available language rendering. **Platform:** verify the import path actually writes multiple `text` rows when given multiple `textContents[]` entries (currently appears single-call).

### 3.4 [HIGH] `electoralPeriod.number` mixes integer-term and year encodings

`electoralPeriod.number` is meant to be the electoral term: the multi-year period between general elections. The local names (Wahlperiode, Legislatura, Législature, 屆, Stortingsperiode, …) are just translations of that one concept, not a real difference. The only structural split is **how the term is encoded**:

| Encoding | Parliaments | Consequence |
|---|---|---|
| Small sequential integer | DE (21) and the German Landtage, ES (15), EU (10), TW (11), FR (17), PT (17), NO (22)¹ | Fits the 3-digit slot; the model's intent |
| 4-digit year | FI (2023), SE (2025) | Overflows the 3-digit slot (§3.1); not numerically comparable to the integer codes |

¹ NO's source is a `"2025-2029"` year-range string, normalised to a synthetic sequential int.

So the field is non-comparable across parliaments for one reason: FI and SE encode it as a year instead of an integer. **SE has a second, deeper problem:** its year is the wrong *level*. The *riksmöte* is an annual session sitting below the 4-year *valperiod* (the actual term), so SE's `number` denotes a sub-period, not the term. FI's 2023 is at least the term itself (the *vaalikausi* start year), just written as a year.

(SE's annual-session choice is one symptom of a broader gap: NO, SE, FI, FR, PT, DE-ST and DE-SH all carry an intra-term level the flat model can't express. Every parliament except SE keeps `electoralPeriod.number` on the term and folds the sub-level into `session.number` or `meta.*`; only SE lets the sub-period become the period number. See §4.2.)

**Fix (Schema):** Promote `electoralPeriod` from `{number: int}` to a richer object:
```json
"electoralPeriod": {
  "number": 16,
  "label": "2022–2026",
  "type": "term",
  "subPeriod": {
    "number": 4,
    "label": "Riksmöte 2025/26",
    "type": "year_range"
  }
}
```
- `type` ∈ `"term" | "year_range" | "session_year"` documents the semantic.
- Optional `subPeriod` accommodates SE's two-level structure (valperiod + riksmöte) and NO's two-level structure (stortingsperiode + sesjon); could carry EU annual sessions later.
- Platform: store `ElectoralPeriodLabel`, `ElectoralPeriodType`; backwards compatible (existing 3-digit number stays).

### 3.5 [MEDIUM] Agenda granularity varies by source, not by parliament

- DE: 1–8 Tagesordnungspunkte per Sitzung, each with multiple speeches; model fits.
- EU: fine-grained, numbered 1..N per day; the model fits and `agendaItem.number` adds useful structure.
- ES: proceedings-side has none; media-side classifier supplies them; the model fits but the source coverage is uneven.
- SE: each agenda item has a cross-session "matter" ID (`HD01UbU20`), which the model's session-scoped agenda granularity loses.
- TW: all speeches in a meeting share one agenda item, and the model expects more granularity than exists.
- DE-SH: a per-speech `(TOP-no, thema)` pair, where one TOP number typically corresponds to one agenda item but each speech carries its own per-turn `thema`. The merger emits one agenda item per speech with `agendaItem.id = "TOP-{n}"`, giving effectively per-speech (not per-TOP) agenda records. Model fits but is slightly over-resolved.
- DE-BY / DE-BW / DE-HH / DE-SN: genuine per-TOP granularity, one `agendaItem` per Tagesordnungspunkt, a clean fit.

**Fix (Schema):** Add optional `agendaItem.originID` (parliament-native stable id). SE populates with matter ID. Platform: add `AgendaItemOriginID` column for de-dup on re-import and cross-session matter linking. Document explicitly that the model accepts coarse agenda (TW = one per meeting) as well as fine (EU = one per item).

### 3.6 [MEDIUM] "Session" is not a unit across parliaments

The label "session" covers different real-world units:

| Parliament | What one Stage 2 session represents |
|---|---|
| DE / DE-RP / DE-ST / DE-SH / DE-BY / DE-BW / DE-HH / DE-NI / DE-NW / DE-SN / ES | one calendar day (one Sitzung / Landtagssitzung / Sesión) |
| EU | one calendar day with 3 sittings collapsed |
| SE | one protokoll (calendar day) with sub-debate "anförande" |
| TW | one meeting within a session period within a term |
| NO | one møte (calendar day), one or more video "del" parts |
| FI | one täysistunto (calendar day), one HLS session video with per-speech offsets |
| FR | one séance (sitting; several per calendar day), one HLS séance video with per-speech offsets |
| PT | one reunião plenária (calendar-day sitting), per-speech server-side HLS clips from one session recording |

**Question / fix (Schema):** Decide whether to add an optional `meta.subSession` field (cheap; lossless for EU sittings and TW meeting decomposition; UI displays it). Alternative: promote to first-class level (`Session > Sitting > AgendaItem > Speech`), which breaks every URL.

Lightweight recommendation: optional `meta.subSession` (free-form string) **and** optional `meta.sourceLabel` (replaces TW's ad-hoc `meta.meetingCode`). No URL change, no schema break.

### 3.7 [LOW] Search filter is German-only

`applyDefaultFilters` in `modules/search/functions.php` excludes 6 hardcoded German title substrings (`"Befragung"`, `"Fragestunde"`, `"Wahl der"`, `"Wahl des"`, `"Sitzungseröffnung"`, `"Sitzungsende"`). For ES, EU, SE, TW these substrings never appear; the filter is a silent no-op. The structural intent (hide procedural items by default) is lost for every parliament except DE.

**Fix (Platform):** Move excluded substrings to per-parliament config. Better: switch from title substrings to `agendaItem.type` / `nativeType` exclusions (e.g. `EU-voting`, `EU-explanations_of_vote`, `EU-procedural`), which is more robust and parliament-agnostic.

---

## 4. Structural questions for architecture review

These can't be patched; they're decisions about what the model should represent.

### 4.1 Should "sitting" be a first-class level between Session and AgendaItem?

EU naturally has 3 sittings per day; TW packs term/session-period/meeting into one field. Today both lose the sub-day structure. The reverse problem also exists: DE-ST has a **super-day** level (Sitzungsperiode = multi-day plenary block) above the canonical day-level Sitzung, currently dropped to `meta.sitzungsperiode`. DE-SH has the same shape under a different name (`Tagung`), currently dropped to `meta.tagung`. Options:
- (A) Optional `meta.subSession` string: cheap, lossless display.
- (B) Promote to schema level (`Session > Sitting > AgendaItem`): breaks URLs.

Recommendation: (A) plus a clearly-named `meta.sourceLabel` to absorb parliament-specific identifiers without per-parliament special fields. DE-ST already populates `meta.sitzungsperiode` (integer) and DE-SH already populates `meta.tagung`; a future `meta.sourceLabel` could subsume both.

### 4.2 Should electoralPeriod be two-level (term + sub-period)?

SE genuinely has two levels (valperiod + riksmöte); NO has two levels (stortingsperiode + sesjon); FI has two levels (vaalikausi + valtiopäivät year, the latter folded into the `session.number` encoding); FR has two levels (législature + session-ordinaire year, folded into `session.number` like FI but with the term kept as a clean integer); PT has two levels (legislatura + sessão legislativa, folded into `session.number` exactly like FR with the term kept integer); DE-ST has two levels (Wahlperiode + Sitzungsperiode, currently dropped); DE-SH has the same shape (Wahlperiode + Tagung); EU and DE could optionally use one (annual sessions of a multi-year term). Today the schema flattens, and SE and FI both encode the period as a year rather than a small integer (SE's year is the riksmöte sub-period, not even the term); FR avoids that by keeping the legislature integer. The `electoralPeriod` schema redesign in §3.4 implements this: once it lands, SE's riksmöte, NO's sesjon, FI's valtiopäivät year, FR's session-ordinaire year, DE-ST's Sitzungsperiode and DE-SH's Tagung can all be promoted to `electoralPeriod.subPeriod.number`.

### 4.3 Should agenda items have stable cross-session identity?

SE uses matter IDs that span sessions; the platform autoincrements per session and loses this. Adding optional `agendaItem.originID` (§3.5) enables cross-session linking without forcing other parliaments to model it.

### 4.4 Should `originPersonID` be a first-class schema field?

SE emits it (Riksdag intressent_id), NO emits it (Stortinget `personID` short codes), DE-NI emits it (Abgeordneten-ID), DE-NW emits it (MdL id). EU has `epId` buried in `additionalInformation`. Bundestag and Yuan have similar identifiers in source data. Useful as a stable cross-check and, for NO, the natural cross-sesjon speaker key. Promotion is additive.

### 4.5 What is the model for parliaments where media ≠ 1:1 with speech?

EU, SE, NO and most of the German Landtage have per-session (or per-debate / per-meeting-part) video with per-speech offsets. Today's schema assumes one MP4 per speech. The `startOffset`/`endOffset` fix in §3.2 is the minimum; the broader question is whether the platform should ever slice videos at import time. Recommendation: no; store offsets and let HLS Media Fragments do the work in the player (the parliaments whose source already serves per-speech clips show this is viable end-to-end).

### 4.6 How are multilingual transcripts represented?

The platform's DB is multilingual-ready; the pipeline is single-language. EU is the urgent case. NO is a secondary case (Bokmål + Nynorsk officially co-equal; current implementation forces `nb-NO`). FI is a third case (Finnish + Swedish officially co-equal): the minutes mark each speech's language, and FI records `originalLanguage` per speech, but runs the pipeline uniformly in Finnish, confirming that `originalLanguage` is cheap to record as data even when per-speech model switching is not built. Recommend: a Tools-side convention of one `textContents[]` entry per available language; platform writes one `text` row each; UI offers a language switcher.

### 4.7 What does the pipeline do for parliaments with no source-side speech enumeration?

There are three regimes, distinguished by which streams carry per-speech identity:

- **Both streams have per-speech identity** (DE, DE-RP, DE-ST, ES, EU, SE, TW, NO, FI, FR, PT). The merger does its normal job.
- **One stream has per-speech identity, the other doesn't** (DE-SH, DE-BY, DE-BW, DE-HH, DE-NI, DE-NW, DE-SN: video ✓, text ✗). The merger still emits Stage 2 from the stream that does, with the other field empty (`textContents: []`). The platform shows clips with speaker / faction / agenda metadata, just no transcript. What we lose by emitting one stream early: search and full-text features degrade; speech-level enrichment and sentence-level alignment are not possible. But the per-speech clip + speaker + agenda metadata is still a valuable platform artefact, and the text half can be added later when a transcript parser exists.
- **Neither stream has per-speech identity** (DE-BE today). No merger can run; the parliament defers until a structured source materialises.

For the third regime, the options are:
- **(a) Accept third-party spine sources** with documented quality caveats and provenance metadata. Cheap to adopt; opens the door to a long tail of regional parliaments. Risk: opaque upstream dependency, hard to validate against truth.
- **(b) Write parliament-specific spine extractors** (PDF parsers, or audio diarization for video-only sources). High quality, full control. Cost: significant effort per parliament.
- **(c) Defer the parliament** until a structured source materialises. Honest but doesn't scale, since every parliament that publishes PDF-only proceedings hits this same wall.

Recommendation: don't pre-decide. The "ship one stream now, add the other later" path also unblocks DE-BE if its video side ever gains per-speech segmentation (currently it doesn't). Build a transcript-source parser when it actually unlocks a parliament that wouldn't otherwise ship.

---

## 5. Recommended sequencing

The work splits into layers with different blast radii:

1. **Schema 1.1, additive only:** §3.2 `media.startOffset/endOffset`, §3.4 `electoralPeriod` as an object with `subPeriod`, §3.5 `agendaItem.originID`, §3.6/§4.1 `meta.subSession` / `meta.sourceLabel`, §4.4 `people[].originPersonID`. Tools populates new fields; Platform import learns to read them (backwards compatible).

2. **EU/SE structural redesign:** §3.1, changing `session.number` semantics for EU and `electoralPeriod` semantics for SE (move to valperiod). Depends on resolving §4.1 (sitting model) and §4.2 (period model). Requires a data re-emission for the affected parliaments.

3. **Platform-side cross-parliament features:** §3.7 per-parliament search filters; §3.3 multi-language `text` import path. Concurrent with parliament rollouts.

A schema 2.0 (breaking) is not currently warranted, since every structural change above can be expressed additively.
