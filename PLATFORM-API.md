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
- **relationships** (properties derived from other data items)
- **links** ("self" = link to the respective API request URL)

Depending on the context, the **attributes** object can only include a subset of all properties. The full set can then be retrieved via an API request to **links** > **self**.

This principle **SHOULD** be applied on all levels of the data structure.

**Example:**  

```yaml
"data": {
  "id": "DE-198765837",
  "type": "media",
  "attributes": {},
  "relationships": {
    "documents": {
      "data": [
        {
          "id": "201",
          "type": "document",
          "attributes": {},
          "links": {
            "self": "https://de.openparliament.tv/api/v1/document/201"
          }
        }
      ],
    }
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
  }
}
```

## 2. Search
`/api/v1/search?params=`  

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
    "additionalInformation": {},
    "aligned": true,
    "audioFileURI": "https://example.com/media.mp3",
    "creator": "Deutscher Bundestag",
    "dateStart": "2007-08-31T16:47+00:00",
    "dateEnd": "2007-08-31T16:58+00:00",
    "duration": 340.32134,
    "lastChanged": "2021-03-18T03:22+00:00",
    "license": "CC-BY-SA",
    "order": 2,
    "originID": "9786865",
    "originMediaID": "7502148",
    "sourcePage": "https://dbtg.tv/fvid/7502148",
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
        "originTextID": "9786865"
      }
    ],
    "thumbnailCreator": "Deutscher Bundestag",
    "thumbnailLicense": "CC-BY-SA",
    "thumbnailURI": "https://example.com/thumb.png",
    "videoFileURI": "https://example.com/media.mp4"
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
            "thumbnailCreator": null,
            "thumbnailLicense": null,
            "thumbnailURI": null
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
            "id": "Q567",
            "attributes": {
              "context": null,
              "type": "party",
              "label": "CDU",
              "thumbnailCreator": "Deutscher Bundestag",
              "thumbnailLicense": "CC-BY-SA",
              "thumbnailURI": "https://example.com/thumb.png",
              "color": "#00ff00"
            },
            "links": {
              "self": "https://de.openparliament.tv/api/v1/organisation/Q567"
            }
          },
          {
            "type": "organisation",
            "id": "Q45345",
            "attributes": {
              "context": null,
              "type": "faction",
              "label": "CDU/CSU",
              "thumbnailCreator": "Deutscher Bundestag",
              "thumbnailLicense": "CC-BY-SA",
              "thumbnailURI": "https://example.com/thumb.png",
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
              "thumbnailCreator": "Deutscher Bundestag",
              "thumbnailLicense": "CC-BY-SA",
              "thumbnailURI": "https://example.com/thumb.png",
              "party": {
                "id": "Q567",
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
  "id": "Q567",
  "type": "person",
  "attributes": {
    "type": "memberOfParliament",
    "label": "Angela Merkel",
    "abstract": "Bundeskanzlerin der Bundesrepublik Deutschland",
    "firstName": "Angela",
    "lastName": "Merkel",
    "degree": "Dr.",
    "gender": "female",
    "birthDate": "1954-07-17",
    "thumbnailURI": "https://example.com/image.png",
    "embedURI": "https://example.com/mobile/Angela_Merkel",
    "websiteURI": "https://www.angela-merkel.de/",
    "socialMediaURIs": [
      {
        "label": "Instagram",
        "url": "instagram.com/Merkel"
      }
    ],
    "originID": "969768675",
    "additionalInformation": {
      "abgeordnetenwatchID": "7643642"
    }
  },
  "relationships": {
    "party": {
      "data": {
        "id": "Q53467",
        "type": "organisation",
        "attributes": {
          "label": "CDU",
          "labelAlternative": "Christlich Demokratische Union",
          "thumbnailURI": "https://example.com/image.png",
          "websiteURI": "https://example.com/"
        },
        "links": {
          "self": "https://de.openparliament.tv/api/v1/organisations/Q53467"
        }
      }
    },
    "faction": {
      "data": {
        "id": "Q98634",
        "type": "organisation",
        "attributes": {
          "label": "CDU/CSU",
          "labelAlternative": "CDU/CSU Fraktion",
          "thumbnailURI": "https://example.com/image.png",
          "websiteURI": "https://example.com/"
        },
        "links": {
          "self": "https://de.openparliament.tv/api/v1/organisations/Q98634"
        }
      }
    },
    "media": {
      "data": [
        {
          "id": "DE-198765837",
          "type": "media",
          "attributes": {}, // TODO: Check which properties are needed here
          "links": {
            "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
          }
        }
      ]
    }
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/person/Q1267"
  }
}
```

### 3.3 Organisation Details
`/api/v1/organisation/ORGANISATIONID`  

**Example Response**  
```yaml
"data": {
  "id": "Q567",
  "type": "organisation",
  "attributes": {},
  "relationships": {
    "media": {
      "data": [
        {
          "id": "DE-198765837",
          "type": "media",
          "attributes": {}, // TODO: Check which properties are needed here
          "links": {
            "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
          }
        }
      ]
    }
  }
  "links": {
    "self": "https://de.openparliament.tv/api/v1/organisation/Q53467"
  }
}
```

### 3.4 Document Details
`/api/v1/document/DOCUMENTID`  

**Example Response**  
```yaml
"data": {
  "id": "3",
  "type": "document",
  "attributes": {},
  "relationships": {
    "media": {
      "data": [
        {
          "id": "DE-198765837",
          "type": "media",
          "attributes": {}, // TODO: Check which properties are needed here
          "links": {
            "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
          }
        }
      ]
    }
  }
  "links": {
    "self": "https://de.openparliament.tv/api/v1/document/3"
  }
}
```

### 3.5 Term Details
`/api/v1/term/TERMID`  

**Example Response**  
```yaml
"data": {
  "id": "165",
  "type": "term",
  "attributes": {},
  "relationships": {
    "media": {
      "data": [
        {
          "id": "DE-198765837",
          "type": "media",
          "attributes": {}, // TODO: Check which properties are needed here
          "links": {
            "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
          }
        }
      ]
    }
  }
  "links": {
    "self": "https://de.openparliament.tv/api/v1/term/165"
  }
}
```

## 4. Platform Statistics
`/api/v1/statistics`  



