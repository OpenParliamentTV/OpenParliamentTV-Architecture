# Platform Data Import

Speeches are imported into the OpenParliamentTV platform from JSON files. The unit per file is **one session** (in live updates the file is rewritten as new speeches are added). Each file holds a `meta` envelope and a `data` array of speech items.

The full format — required and optional fields, text and video modes, producer obligations — is defined in **[STAGE2-FORMAT.md](STAGE2-FORMAT.md)**, with machine-readable schemas in [Tools/optv/shared/schema/](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/tree/main/optv/shared/schema).

For end-to-end pipeline context, see [PIPELINE.md](PIPELINE.md).
