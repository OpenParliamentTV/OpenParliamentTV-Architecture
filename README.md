# OpenParliamentTV Architecture

OpenParliamentTV is a video platform for parliamentary debates: every speech is searchable through its transcript, and every transcript sentence can jump the player to the matching point in the video. This repository is the cross-repo home for the project's specifications, conventions, and architecture documentation.

## Specs

- [PIPELINE.md](PIPELINE.md) — How parliament data flows from source to platform: Stage 1 (parliament-specific fetch/parse/merge) → Stage 2 (common format) → Stage 3+ (enrichment, validation, publishing).
- [STAGE2-FORMAT.md](STAGE2-FORMAT.md) — The Stage 2 JSON format every parliament implementation must produce. Field categories, text and video modes, producer obligations.
- [SHORTCODES.md](SHORTCODES.md) — Naming conventions for instances, parliaments, and languages.
- [PLATFORM-API.md](PLATFORM-API.md) — Public JSON:API specification.
- [PLATFORM-DATAOBJECTS.md](PLATFORM-DATAOBJECTS.md) — Example responses for the platform's data types.
- [PLATFORM-DATAIMPORT.md](PLATFORM-DATAIMPORT.md) — Platform import contract; defers field-level details to STAGE2-FORMAT.md.
- [PLATFORM-URLS.md](PLATFORM-URLS.md) — Frontend and API URL patterns.

## Repositories

```
   ┌──────────────────────────┐
   │    Parliament sources    │  TEI XML, Akoma Ntoso, RSS, custom APIs
   └────────────┬─────────────┘
                │
                ▼
   ┌──────────────────────────┐
   │   OpenParliamentTV-Tools │  Stage 1 (per parliament) + Stage 2 (shared)
   │   (run by Conductor)     │  fetch · parse · merge · NEL · align · NER
   └────────────┬─────────────┘
                │  Stage 2 JSON
                ▼
   ┌──────────────────────────┐         ┌────────────────────────────────────┐
   │  OpenParliamentTV-Data-* │         │ OpenParliamentTV-                  │
   │  (per-parliament repos)  │         │ Additional-Data-Service            │
   └────────────┬─────────────┘         │ Wikidata · Wikipedia · Commons ·   │
                │                       │ + per-parliament sources           │
                ▼                       └────────────────┬───────────────────┘
   ┌──────────────────────────┐                          │
   │ OpenParliamentTV-Platform│ ◀────── entity enrichment ┘
   │   (public site & API)    │
   └──────────────────────────┘
```

| Repository | Role |
|------------|------|
| [OpenParliamentTV-Architecture](https://github.com/OpenParliamentTV/OpenParliamentTV-Architecture) | This repo. Specs, conventions, cross-repo wiring. |
| [OpenParliamentTV-Tools](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools) | The import pipeline. Parliament-specific scrapers/parsers/mergers (currently: German Bundestag) plus shared NEL, alignment, NER, validation, and publishing. |
| [OpenParliamentTV-Conductor](https://github.com/OpenParliamentTV/OpenParliamentTV-Conductor) | Web-based orchestrator. Runs Tools per parliament: queues jobs, schedules recurring runs, streams logs. |
| [OpenParliamentTV-Platform](https://github.com/OpenParliamentTV/OpenParliamentTV-Platform) | Public site and API. Consumes the per-parliament data repos. |
| [OpenParliamentTV-Additional-Data-Service](https://github.com/OpenParliamentTV/OpenParliamentTV-Additional-Data-Service) | REST API the platform calls at runtime to enrich entity profiles (people, organisations, documents, terms) by aggregating Wikidata, Wikipedia, Wikimedia Commons, and per-parliament sources. Includes optional response caching. |
| `OpenParliamentTV-Data-<CODE>` | One repo per parliament instance, holding the raw and processed Stage 2 JSON files. Example: [OpenParliamentTV-Data-DE](https://github.com/OpenParliamentTV/OpenParliamentTV-Data-DE). |

## Adding a parliament

The end-to-end walkthrough lives in Tools, alongside the code: [Tools/docs/ADDING-A-PARLIAMENT.md](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/blob/main/docs/ADDING-A-PARLIAMENT.md).
