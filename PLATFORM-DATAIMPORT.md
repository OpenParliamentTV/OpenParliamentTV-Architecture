# Open Parliament Platform Data Import (DRAFT)

### 1. General Notes

Speeches are imported into the platform from JSON files. The unit for these files is **1 session** (in the case of live updates this would contain the number of speeches already held and is continously updated). 

Each of these JSON files contains an **Array** of **Speech Objects**.

### 2. Sample / Blueprint for a single Speech Object 

```yaml
{
  "parliament": "DE",
  "electoralPeriod": {
    "number": 19
  },
  "session": {
    "number": 110,
    "dateStart": "2018-03-18T00:00+00:00",
    "dateEnd": "2018-03-18T00:00+00:00"
  },
  "agendaItem": {
    "officialTitle": "Tagesordnungspunkt 1",
    "title": "Bundeswehreinsatz in Afghanistan"
  },
  "media": {
    "audioFileURI": "https://example.com/media.mp3",
    "videoFileURI": "https://example.com/media.mp4",
    "thumbnailURI": "https://example.com/thumb.png",
    "thumbnailCreator": "Deutscher Bundestag",
    "thumbnailLicense": "CC-BY-SA",
    "duration": 340.32134,
    "creator": "Deutscher Bundestag",
    "license": "CC-BY-SA",
    "originID": "6745754",
    "originMediaID": "7502148",
    "sourcePage": "https://dbtg.tv/fvid/7502148"
  },
  "textContents": [
    {
      "type": "proceedings",
      "textBody": [
        {
            "type": "speech",
            "speaker": "Angela Merkel",
            "text": "Sehr geehrte Damen und Herren, ..."
        },
        {
            "type": "comment",
            "speaker": null,
            "text": "(Beifall bei der CDU/CSU, der FDP und dem BÜNDNIS 90/DIE GRÜNEN sowie bei Abgeordneten der SPD und der AfD)"
        }
      ],
      "sourceURI": "https://www.bundestag.de/resource/blob/813c4/19203-data.xml",
      "creator": "Deutscher Bundestag",
      "license": "Public Domain",
      "language": "DE-de",
      "originTextID": "9786865"
    }
  ],
  "people": [
    {
      "type": "memberOfParliament",
      "label": "Angela Merkel",
      "context": "mainSpeaker",
      "faction": "CDU/CSU",
      "party": "CDU"
    },
    {
      "type": "memberOfParliament",
      "label": "Wolfgang Schäuble",
      "context": "president"
    }
  ],
  "documents": [
    {
      "type": "officialDocument",
      "label": "Drucksache 19/2",
      "sourceURI": "https://dserver.bundestag.de/btd/19/000/1900002.pdf"
    }
  ],
  "dateStart": "2018-08-31T16:47+00:00",
  "dateEnd": "2018-08-31T16:58+00:00"
}
```
