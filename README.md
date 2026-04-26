# Open Parliament TV - Architecture

**Open Parliament TV** is a **search engine** and **interactive video platform** for **parliamentary debates**. This repository is the cross-repo home for the project's specifications, conventions, and architecture documentation.

## Specs

- [PIPELINE.md](PIPELINE.md) — How parliament data flows from source to platform: Stage 1 (parliament-specific fetch/parse/merge) → Stage 2 (common format) → Stage 3+ (enrichment, validation, publishing).
- [STAGE2-FORMAT.md](STAGE2-FORMAT.md) — The Stage 2 JSON format every parliament implementation must produce. Field categories, text and video modes, producer obligations.
- [SHORTCODES.md](SHORTCODES.md) — Naming conventions for instances, parliaments, and languages.
- [PLATFORM-API.md](PLATFORM-API.md) — Public JSON:API specification.
- [PLATFORM-DATAOBJECTS.md](PLATFORM-DATAOBJECTS.md) — Example responses for the platform's data types.
- [PLATFORM-URLS.md](PLATFORM-URLS.md) — Frontend and API URL patterns.

## Repositories

![OpenParliamentTV architecture](OpenParliamentTV-Architecture.svg)

| Repository | Role |
|------------|------|
| [OpenParliamentTV-Architecture](https://github.com/OpenParliamentTV/OpenParliamentTV-Architecture) | This repository. Specs, conventions, general architecture. |
| [OpenParliamentTV-Tools](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools) | The import pipeline. Parliament-specific scrapers/parsers/mergers plus shared NEL, alignment, NER, validation, and publishing. |
| [OpenParliamentTV-Conductor](https://github.com/OpenParliamentTV/OpenParliamentTV-Conductor) | Web-based orchestrator. Runs [OpenParliamentTV-Tools](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools) per parliament: queues jobs, schedules recurring runs, streams logs. |
| `OpenParliamentTV-Data-<CODE>` | One repository per parliament instance, holding the raw and processed Stage 2 JSON files. Example: [OpenParliamentTV-Data-DE](https://github.com/OpenParliamentTV/OpenParliamentTV-Data-DE). |
| [OpenParliamentTV-Platform](https://github.com/OpenParliamentTV/OpenParliamentTV-Platform) | Open Parliament TV platform instances. Consume the per-parliament data repositories. |
| [OpenParliamentTV-Additional-Data-Service](https://github.com/OpenParliamentTV/OpenParliamentTV-Additional-Data-Service) | REST API the platform calls at runtime to enrich entity profiles (people, organisations, documents, terms) by aggregating Wikidata, Wikipedia, Wikimedia Commons, and per-parliament sources. |

## Adding a parliament

The end-to-end walkthrough lives in [OpenParliamentTV-Tools](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools), alongside the code: [Tools/docs/ADDING-A-PARLIAMENT.md](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/blob/main/docs/ADDING-A-PARLIAMENT.md).
