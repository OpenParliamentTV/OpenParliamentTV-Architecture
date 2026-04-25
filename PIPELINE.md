# Pipeline

Parliamentary data flows through a staged pipeline:

```
Stage 1                    Stage 2                       Stage 3+
─────────────────────      ───────────────────────       ─────────────────────
Parliament-specific        Common format                 Enrichment & delivery
fetch + parse + merge  →   one JSON file per session  →  validate, publish,
(per parliament)           (shared infrastructure)       index, etc.
```

The pipeline is implemented in [OpenParliamentTV-Tools](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools). [OpenParliamentTV-Conductor](https://github.com/OpenParliamentTV/OpenParliamentTV-Conductor) is the web-based orchestrator that runs Tools per parliament.

To add a new parliament, see [Tools/docs/ADDING-A-PARLIAMENT.md](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/blob/main/docs/ADDING-A-PARLIAMENT.md).

---

## Stage 1 — Parliament-specific

Each parliament has its own scrapers, parsers, and merger, because every parliament publishes its data differently (TEI XML, Akoma Ntoso, ParlaMint, custom JSON/HTML APIs, RSS feeds, …).

| Step | Job |
|------|-----|
| **Fetch** | Download raw proceedings (transcripts) and media (RSS / metadata feeds) from the parliament's open-data sources. |
| **Parse** | Convert the parliament's native format into intermediate JSON. |
| **Merge** | Join the proceedings stream with the media stream into one record per speech, producing a Stage 2 JSON file per session. |

Stage 1 ends with a Stage 2 file. The format is defined in [STAGE2-FORMAT.md](STAGE2-FORMAT.md) and validated by the schemas in [Tools/optv/shared/schema/](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/tree/main/optv/shared/schema).

---

## Stage 2 — Shared enrichment

Once data is in the common Stage 2 shape, the pipeline runs parliament-agnostic stages:

| Stage | What it does |
|-------|--------------|
| **NEL** | Named-entity linking for people and factions: resolves `people[].wid` against Wikidata, normalises faction shape. Skipped when source already provides Wikidata IDs. |
| **Align** | Sentence-level forced alignment of transcript text against audio (currently via [aeneas](https://github.com/readbeyond/aeneas)). Produces `timeStart` / `timeEnd` per sentence. Skipped when source already provides timings. |
| **NER** | Named-entity recognition over speech text (spaCy + entity-fishing). Produces `entities[]` per sentence. |

Each stage is idempotent — it fills in only the fields that are missing — so the same code path serves both live ingest and historical-corpus imports.

---

## Stage 3+ — Validate, publish, deliver

| Stage | What it does |
|-------|--------------|
| **Validate** | Run [`validate_stage2()`](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/blob/main/optv/shared/validators/) against the full schema and the semantic-validator rules. Findings are logged; warnings do not block publishing. |
| **Publish** | Copy the latest cache file to the per-parliament data repo (`OpenParliamentTV-Data-<CODE>`), commit, and push. The platform consumes from these repos. |

Beyond publishing, the platform itself runs further enrichment (search indexing, speaker-cache updates, Wikidata cross-lookups). These are platform concerns, not part of the import pipeline, and are not specified here.

---

## Orchestration

Tools provides the stage implementations as a Python package and a per-parliament `workflow.py` entry point. Conductor wraps that in a web UI: queues jobs, schedules recurring runs, streams logs, surfaces failures. The two repos remain separate because Tools is the data contract and Conductor is operational infrastructure — Tools is usable on its own from the command line.
