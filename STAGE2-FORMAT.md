# Stage 2 Format

Stage 2 is the parliament-agnostic JSON format that every parliament implementation produces. It is the contract between the parliament-specific scraping/parsing code and the platform that consumes the data.

The machine-readable schema lives in [OpenParliamentTV-Tools/optv/shared/schema/](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/tree/main/optv/shared/schema):

- [`stage2-minimal.schema.json`](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/blob/main/optv/shared/schema/stage2-minimal.schema.json) — the minimum a file must contain to be importable by the platform.
- [`stage2-full.schema.json`](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/blob/main/optv/shared/schema/stage2-full.schema.json) — every field the pipeline knows about.

A schema-by-schema reference (datetime patterns, Wikidata patterns, enum values, deprecated fields) is in [optv/shared/schema/README.md](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/blob/main/optv/shared/schema/README.md). Real-world examples are in [optv/shared/docs/EXAMPLES/](https://github.com/OpenParliamentTV/OpenParliamentTV-Tools/tree/main/optv/shared/docs/EXAMPLES).

This document describes the format conceptually: envelope, field categories, text and video modes.

---

## Principles

- **Video is required.** OPTV is a video platform. A parliament without accessible video is not a viable target, regardless of text quality.
- **One pipeline for historical and live data.** Pre-aligned corpora skip alignment; pre-resolved Wikidata IDs skip NEL. The pipeline is idempotent and only fills in what is missing.
- **One unit per file: a session.** Each file holds the speeches of one session. Live updates rewrite the file as new speeches are added.

---

## Envelope

```json
{
  "meta": {
    "session": "20001",
    "dateStart": "2021-10-26T09:00:00+02:00",
    "dateEnd":   "2021-10-26T18:30:00+02:00",
    "processing": {
      "parse_media":       "2024-01-15T10:00:00",
      "parse_proceedings": "2024-01-15T10:01:00",
      "merge":             "2024-01-15T10:02:00"
    },
    "schemaVersion": "1.0"
  },
  "data": [ /* speech items */ ]
}
```

`meta.schemaVersion` is optional; an absent value means `"1.0"`. `meta.processing.<stage>` timestamps are written by the pipeline as each stage runs.

---

## Required fields (minimal schema)

These are the fields the platform must see to accept a speech:

| Field | Type | Notes |
|-------|------|-------|
| `parliament` | string | Shortcode from [SHORTCODES.md](SHORTCODES.md) (e.g. `DE`, `DE-BE`). |
| `electoralPeriod.number` | integer | Legislative term. |
| `session.number` | integer | Session number within the term. |
| `agendaItem.officialTitle` | string | Official agenda item title. |
| `agendaItem.title` | string | Human-readable agenda topic. |
| `dateStart` | ISO 8601 | Speech start with timezone. |
| `media.videoFileURI` | URL | Direct link to the video file. |
| `media.sourcePage` | URL | Source page on the parliament's site. |
| `people[].label` | string | Speaker display name. |
| `people[].context` | enum | See valid values in the schema README. |
| `people[].wid` | string | Wikidata QID. May be empty; the NEL stage fills it later (warning, not an error). |

---

## Platform-important fields

Not strictly required, but core platform features depend on them. Producers should populate these whenever the source provides them.

| Field | Why it matters | Source / derivable |
|-------|----------------|--------------------|
| `textContents[]` | Full-text search, transcript display, sentence-level video sync. See [Text content](#text-content) below. | From the proceedings/transcript source. |
| `people[].faction` | Filter by faction; faction colour-coding. | Best from source; can be looked up from Wikidata via the speaker's `wid` + period, but fragile. |
| `dateEnd` | Speech duration, timeline navigation. | Computable from `dateStart` + `media.duration`, or from the next speech. |
| `media.duration` | Progress bar, duration display. | Pipeline can derive via ffprobe if omitted. |
| `media.creator` | Attribution. | Often a per-parliament constant. |
| `media.license` | Legal use & redistribution. | Often a per-parliament constant. |

---

## Pipeline-managed fields

Set by the pipeline itself; producers do not author these.

| Field | Set by |
|-------|--------|
| `speechIndex` | Assigned from speech order. |
| `media.aligned` | Set after the alignment stage. |
| `meta.processing.*` | Each pipeline stage stamps its own timestamp. |
| `debug.*` | Confidence scores, merge cross-references, stage durations. |

---

## Optional fields

Add value but not blocking: `originTextID`, `media.audioFileURI`, `media.originMediaID`, `media.additionalInformation` (free-form JSON for parliament-specific media metadata), `media.thumbnailURI` and its creator/license, `people[].role`, `documents[]` (referenced bills, Drucksachen, etc.).

---

## Text content

`textContents` is the most important non-required field. A speech with only video and metadata is a clip with metadata; merging video with structured, searchable transcript text is what makes OPTV distinctive.

Text is supported in two modes:

1. **Aligned** — `sentences[]` carry `timeStart` / `timeEnd`. Enables the synchronised transcript: clicking a sentence jumps the player; the current sentence highlights during playback.
2. **Non-aligned** — `sentences[]` omit timing (or the array is absent and only paragraph `text` is present). The transcript is still indexed and searchable; only sentence-level sync is unavailable.

Producers should always include `textContents` when transcript text exists, with or without timings. The pipeline's alignment stage can add timings later.

```json
{
  "textContents": [{
    "type": "proceedings",
    "language": "DE-de",
    "originTextID": "ID19001-123",
    "sourceURI": "https://bundestag.de/...",
    "creator": "Deutscher Bundestag",
    "license": "Public Domain",
    "textBody": [{
      "type": "speech",
      "speaker": "Max Mustermann",
      "speakerstatus": "main-speaker",
      "text": "<p data-type=\"speech\">Full paragraph text here...</p>",
      "sentences": [
        { "text": "First sentence.",  "timeStart": "1.000", "timeEnd": "3.120" },
        { "text": "Second sentence.", "timeStart": "3.120", "timeEnd": "4.960" }
      ]
    }]
  }]
}
```

---

## Video reference modes

The format always operates per speech: every entry in `data[]` is one speech with its own `media.videoFileURI`. How that URI is constructed differs between parliaments:

- **Per-speech video file** — each speech has its own dedicated file (e.g. German Bundestag, Swedish Riksdagen).
- **Session-level video with timecodes** — one video file covers the whole session, and each speech's `videoFileURI` includes a media-fragment range (e.g. `…/session.mp4#t=120,596`) or a parliament-specific query parameter. The data model is unchanged; the producer is responsible for computing the timecodes.

`media.additionalInformation` (free-form JSON object) can carry supplementary metadata such as the un-fragmented session URL when the per-speech URI uses fragments.

### Alignment requirements

For session-level video without source-provided sentence timings, the alignment stage needs isolated audio per speech to run forced alignment. The producer must either:

1. Extract per-speech audio segments from the session video using speech boundary times, then run alignment over them, or
2. Fetch per-speech clips from a parliament-provided clipping service.

If sentence timings are already in the source data, alignment is skipped entirely.

---

## Speaker resolution

If the source already provides a Wikidata ID (e.g. ParlaMint `<idno type="wikimedia">`, Riksdagen API), pass it through. Otherwise leave `people[].wid` empty — the NEL stage will resolve it against the platform's growing speaker cache. There is no need to maintain a parliament-specific speaker cache; the platform's NEL infrastructure handles this once a speaker has been added anywhere.

A missing `wid` is a validation warning, never a blocker.

---

## What a producer must deliver

Required:

- Parliament code, electoral period, session number
- Agenda item (at least one of `officialTitle` / `title`; in practice both)
- `dateStart` with timezone
- `media.videoFileURI` (per-speech or session+timecodes)
- `media.sourcePage`
- At least one speaker with `label` and `context`

Should-deliver-when-possible:

- `textContents` (aligned or not)
- `people[].faction`
- `dateEnd`
- `media.creator`, `media.license`

Pipeline-managed (do not author): `speechIndex`, `media.duration`, `media.aligned`, `meta.processing.*`, `debug.*`.
