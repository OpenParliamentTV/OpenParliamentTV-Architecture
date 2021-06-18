# Open Parliament Platform Public API (DRAFT)

## 1. General Notes

For now we will roughly follow the [JSON:API Specification](https://jsonapi.org/format/). Whether or not we will fully implement the standard (also for PATCH requests / data updates) is to be discussed.

### 1.1 Paths
**API Documentation**  
`/api`  

**API Endpoint Base URL**  
`/api/v1`  

### 1.2 API Responses

Responses **MUST** include the following properties for any request (GET **and** POST):

```yaml
{
  "meta": {
    "api": {
      "version": "1.0",
      "documentation": "https://de.openparliament.tv/api"
    },
    "requestStatus": "success" // OR "error"
  },
  "data": [], // {} OR []
  "errors": [], // EITHER "data" OR "errors"
  "links": {
    "self": "https://de.openparliament.tv/api/v1/search?q=Rente" // request URL
  }
}
```

**Successful** requests **MUST** include the following properties:

```yaml
{
  "meta": {
    "api": {
      "version": "1.0",
      "documentation": "https://de.openparliament.tv/api"
    },
    "requestStatus": "success",
    "code": "2",
    "detail": "Request successfull"
  },
  "data": {},
  "links": {
    "self": "https://de.openparliament.tv/api/v1/search?q=Rente" // request URL
  }
}
```

**Errors** **MUST** include the following properties:

```yaml
{
  "meta": {
    "api": {
      "version": "1.0",
      "documentation": "https://de.openparliament.tv/api"
    },
    "requestStatus": "error"
  },
  "errors": [
    {
      "meta": {
        "domSelector": "" // optional
      },
      "status": "422", // HTTP Status   
      "code": "3", 
      "title":  "Invalid Attribute",
      "detail": "First name must contain at least three characters."
    }
  ],
  "links": {
    "self": "https://de.openparliament.tv/api/v1/search?q=Rente" // request URL
  }
}
```

### 1.3 Data Objects

Data Objects **MUST** always include the properties:  
- **id**
- **type**
- **attributes** (data item specific properties)

Additionally Data Objects **CAN** include the properties:
- **links** ("self" = link to the respective API request URL)
- **relationships** (properties derived from other data items)

Depending on the context, the **attributes** object can only include a subset of all properties. The full set can then be retrieved via an API request to **links** > **self**.

This principle **SHOULD** be applied on all levels of the data structure.

**Example:**  

```yaml
"data": {
  "type": "media",
  "id": "DE-198765837",
  "attributes": {},
  "links": {
    "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
  },
  "relationships": {
    "documents": {
      "data": [
        {
          "type": "document",
          "id": "201",
          "attributes": {},
          "links": {
            "self": "https://de.openparliament.tv/api/v1/document/201"
          }
        }
      ],
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchAnnotations?mediaID=DE-198765837&type=document"
      }
    }
  }
}
```

## 2. Search
`/api/v1/searchMedia?params=`  
`/api/v1/searchPeople?params=`  
`/api/v1/searchOrganisations?params=`  
`/api/v1/searchDocuments?params=`  
`/api/v1/searchTerms?params=`  
`/api/v1/searchAnnotations?params=`  


**Parameters**  
|Parameter |Notes  |
--- | ---
|param1|Explaination|
|param2|Explaination|

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```

## 3. Data Object Details
### 3.1 Media Details
`/api/v1/media/MEDIAID`  

**Example Response**  
```yaml
"data": {
  "type": "media",
  "id": "DE-198765837",
  "attributes": {
    "originID": "9786865",
    "originMediaID": "7502148",
    "creator": "Deutscher Bundestag",
    "license": "CC-BY-SA",
    "order": 2,
    "aligned": true,
    "dateStart": "2007-08-31T16:47+00:00",
    "dateEnd": "2007-08-31T16:58+00:00",
    "duration": 340.32134,
    "videoFileURI": "https://example.com/media.mp4"
    "audioFileURI": "https://example.com/media.mp3",
    "sourcePage": "https://dbtg.tv/fvid/7502148",
    "thumbnailURI": "https://example.com/thumb.png"
    "thumbnailCreator": "Deutscher Bundestag",
    "thumbnailLicense": "CC-BY-SA",
    "additionalInformation": {},
    "lastChanged": "2021-03-18T03:22+00:00",
    "textContents": [
      {
        "id": 34,
        "type": "text",
        "textBody": [
          {
            "type": "speech",
            "text": "Sehr geehrte Damen und Herren, ..."
          },
          {
            "type": "comment",
            "text": "(Beifall bei der CDU/CSU, der FDP und dem BÜNDNIS 90/DIE GRÜNEN sowie bei Abgeordneten der SPD und der AfD)"
          }
        ],
        "sourceURI": "https://www.bundestag.de/resource/blob/813c4/19203-data.xml",
        "creator": "Deutscher Bundestag",
        "license": "Public Domain",
        "language": "DE-de",
        "originTextID": "9786865",
        "lastChanged": "2021-03-18T03:22+00:00"
      }
    ]
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
  },
  "relationships": {
    "electoralPeriod": {
      "data": {
        "type": "electoralPeriod",
        "id": 3,
        "attributes": {
          "number": 19
        },
        "links": {
          "self": "https://de.openparliament.tv/api/v1/electoralperiod/DE-19"
        }
      }
    },
    "session": {
      "data": {
        "type": "session",
        "id": 84,
        "attributes": {
            "number": 110
        },
        "links": {
          "self": "https://de.openparliament.tv/api/v1/session/DE-110"
        }
      }
    },
    "agendaItem": {
      "data": {
        "type": "agendaItem",
        "id": 3,
        "attributes": {
          "officialTitle": "Tagesordnungspunkt 1",
          "title": "Bundeswehreinsatz in Afghanistan"
        },
        "links": {
          "self": "https://de.openparliament.tv/api/v1/agendaitem/DE-3"
        }
      }
    },
    "documents": {
      "data": [
        {
          "type": "document",
          "id": 201,
          "attributes": {
            "context": null,
            "type": "officialDocument",
            "label": "Drucksache 12/2",
            "labelAlternative": "12/2",
            "thumbnailURI": null
            "thumbnailCreator": null,
            "thumbnailLicense": null,
          },
          "links": {
            "self": "https://de.openparliament.tv/api/v1/document/201"
          }
        }
      ],
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchAnnotations?mediaID=DE-198765837&type=document"
      }
    },
    "organisations": {
        "data": [
          {
            "type": "organisation",
            "id": "Q356465",
            "attributes": {
              "context": null,
              "type": "party",
              "label": "CDU",
              "alternativeLabel": "Christlich Demokratische Union",
              "thumbnailURI": "https://example.com/thumb.png",
              "thumbnailCreator": "Deutscher Bundestag",
              "thumbnailLicense": "CC-BY-SA",
              "color": "#00ff00"
            },
            "links": {
              "self": "https://de.openparliament.tv/api/v1/organisation/Q356465"
            }
          },
          {
            "type": "organisation",
            "id": "Q45345",
            "attributes": {
              "context": null,
              "type": "faction",
              "label": "CDU/CSU",
              "alternativeLabel": "CDU/CSU Fraktion",
              "thumbnailURI": "https://example.com/thumb.png",
              "thumbnailCreator": "Deutscher Bundestag",
              "thumbnailLicense": "CC-BY-SA",
              "color": "#ff0000"
            },
            "links": {
              "self": "https://de.openparliament.tv/api/v1/organisation/Q45345"
            }
          }
        ],
        "links": {
            "self": "https://de.openparliament.tv/api/v1/searchAnnotations?mediaID=DE-198765837&type=person"
        }
    },
    "people": {
        "data": [
          {
            "type": "person",
            "id": "Q2889",
            "attributes": {
              "context": "speaker",
              "type": "memberOfParliament",
              "label": "Angela Merkel",
              "degree": "Dr.",
              "thumbnailURI": "https://example.com/thumb.png",
              "thumbnailCreator": "Deutscher Bundestag",
              "thumbnailLicense": "CC-BY-SA",
              "party": {
                "id": "Q356465",
                "label": "CDU"
              },
              "faction": {
                "id": "Q45345",
                "label": "CDU/CSU"
              }
            },
            "links": {
              "self": "https://de.openparliament.tv/api/v1/person/Q2889"
            }
          }
        ],
        "links": {
            "self": "https://de.openparliament.tv/api/v1/searchAnnotations?mediaID=DE-198765837&type=person"
        }
    },
    "annotations": {
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchAnnotations?mediaID=DE-198765837"
      }
    },
    "terms": {
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchAnnotations?mediaID=DE-198765837&type=terms"
      }
    }
  }
}
```

### 3.2 Person Details
`/api/v1/person/PERSONID`  

**Example Response**  
```yaml
"data": {
  "type": "person",
  "id": "Q567",
  "attributes": {
    "type": "memberOfParliament",
    "label": "Angela Merkel",
    "firstName": "Angela",
    "lastName": "Merkel",
    "degree": "Dr.",
    "birthDate": "1954-07-17",
    "gender": "female",
    "abstract": "Bundeskanzlerin der Bundesrepublik Deutschland",
    "thumbnailURI": "https://example.com/image.png",
    "thumbnailCreator": "Deutscher Bundestag",
    "thumbnailLicense": "CC-BY-SA",
    "embedURI": "https://example.com/mobile/Angela_Merkel",
    "websiteURI": "https://www.angela-merkel.de/",
    "originID": "969768675",
    "socialMediaIDs": [
      {
        "label": "Instagram",
        "id": "Merkel"
      }
    ],
    "additionalInformation": {
      "abgeordnetenwatchID": "7643642"
    },
    "lastChanged": "2021-03-18T03:22+00:00"
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/person/Q1267"
  },
  "relationships": {
    "party": {
      "data": {
        "type": "organisation",
        "id": "Q53467",
        "attributes": {
          "label": "CDU",
          "labelAlternative": "Christlich Demokratische Union",
          "thumbnailURI": "https://example.com/image.png",
          "thumbnailCreator": "Deutscher Bundestag",
          "thumbnailLicense": "CC-BY-SA",
          "websiteURI": "https://example.com/"
        },
        "links": {
          "self": "https://de.openparliament.tv/api/v1/organisations/Q53467"
        }
      }
    },
    "faction": {
      "data": {
        "type": "organisation",
        "id": "Q98634",
        "attributes": {
          "label": "CDU/CSU",
          "labelAlternative": "CDU/CSU Fraktion",
          "thumbnailURI": "https://example.com/image.png",
          "thumbnailCreator": "Deutscher Bundestag",
          "thumbnailLicense": "CC-BY-SA",
          "websiteURI": "https://example.com/"
        },
        "links": {
          "self": "https://de.openparliament.tv/api/v1/organisations/Q98634"
        }
      }
    },
    "media": {
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchMedia?personID=Q1267"
      }
    }
  }
}
```

### 3.3 Organisation Details
`/api/v1/organisation/ORGANISATIONID`  

**Example Response**  
```yaml
"data": {
  "type": "organisation",
  "id": "Q53467",
  "attributes": {
    "type": "party",
    "label": "CDU",
    "labelAlternative": "Christlich Demokratische Union",
    "abstract": "Partei in Deutschland",
    "thumbnailURI": "https://example.com/image.png",
    "thumbnailCreator": "CDU Deutschland",
    "thumbnailLicense": "CC-BY-SA",
    "embedURI": null,
    "websiteURI": "https://cdu.de",
    "socialMediaIDs": [
      {
        "label": "Twitter",
        "id": "CDU"
      },
      {
        "label": "Facebook",
        "id": "CDU"
      },
      {
        "label": "Instagram",
        "id": "cdu"
      }
    ],
    "color": "#0000ff",
    "additionalInformation": {},
    "lastChanged": "2021-03-18T03:22+00:00"
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/organisation/Q53467"
  },
  "relationships": {
    "media": {
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchMedia?organisationID=Q53467"
      }
    },
    "people": {
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchPeople?organisationID=Q53467"
      }
    }
  }
}
```

### 3.4 Document Details
`/api/v1/document/DOCUMENTID`  

**Example Response**  
```yaml
"data": {
  "type": "document",
  "id": "3",
  "attributes": {
    "type": "officialDocument",
    "wikidataID": null,
    "label": "Drucksache 12/2",
    "labelAlternative": "12/2",
    "abstract": "",
    "thumbnailURI": "https://example.com/image.png",
    "thumbnailCreator": "system",
    "thumbnailLicense": "Public Domain",
    "sourceURI": "https://dserver.bundestag.de/btd/19/000/1900002.pdf"
    "embedURI": null,
    "additionalInformation": {},
    "lastChanged": "2021-03-18T03:22+00:00"
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/document/3"
  },
  "relationships": {
    "media": {
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchMedia?documentID=3"
      }
    }
  }
}
```

### 3.5 Term Details
`/api/v1/term/TERMID`  

**Example Response**  
```yaml
"data": {
  "type": "term",
  "id": "165",
  "attributes": {
    "type": null,
    "wikidataID": null,
    "label": "Subsidiaritätsprinzip",
    "labelAlternative": null,
    "abstract": null,
    "thumbnailURI": "https://example.com/image.png",
    "thumbnailCreator": "system",
    "thumbnailLicense": "Public Domain",
    "sourceURI": null,
    "embedURI": null,
    "additionalInformation": {},
    "lastChanged": "2021-03-18T03:22+00:00"
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/term/165"
  },
  "relationships": {
    "media": {
      "links": {
        "self": "https://de.openparliament.tv/api/v1/searchMedia?termID=165"
      }
    }
  }
}
```

## 4. Platform Statistics
`/api/v1/statistics`  



