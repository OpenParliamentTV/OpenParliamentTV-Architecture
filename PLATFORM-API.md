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

Examples for all data types see: [PLATFORM-DATAOBJECTS.md](PLATFORM-DATAOBJECTS.md)

## 2. Search

##### `/api/v1/search/media?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| q | min 3 chars | Full Text Search | String |
| party | min 1 char | label, labelAlternative | String OR Array |
| partyID | min 1 char | partyOrganisationID | String OR Array |
| faction | min 1 char | label, labelAlternative | String OR Array |
| factionID | min 1 char | partyOrganisationID | String OR Array |
| name | min 3 chars | ??? | String |
| ... |  | ... | String OR Array |

##### `/api/v1/search?type=people`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| name | min 3 chars | label, firstName, lastName | String |
| type | min 1 char | type | String |
| party | min 1 char | organisation.label, organisation.labelAlternative | String OR Array |
| partyID | min 1 char | partyOrganisationID | String OR Array |
| faction | min 1 char | organisation.label, organisation.labelAlternative | String OR Array |
| factionID | min 1 char | factionOrganisationID | String OR Array |
| degree | min 1 char | degree | String |
| gender | min 1 char | gender | String |
| originID | min 1 char | originID | String |
| abgeordnetenwatchID | min 1 char | additionalInformation.abgeordnetenwatchID | String |


##### `/api/v1/search/organisations?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| name | min 3 chars | label, labelAlternative | String OR Array |
| type | min 1 char | type | String |

##### `/api/v1/search/documents?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| label | min 3 chars | label, labelAlternative | String OR Array |
| type | min 1 char | type | String |
| wikidataID | min 1 char | wikidataID | String |

##### `/api/v1/search/terms?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| label | min 3 chars | label, labelAlternative | String OR Array |
| wikidataID | min 1 char | wikidataID | String |

___

**Example Response**  
```yaml
{
  "meta": {
    "api": {
      "version": "1.0",
      "documentation": "https://de.openparliament.tv/api"
    },
    "requestStatus": "success",
    "results": {
      "count": 25,
      "total": 128,
      "rangeStart": 50,
      "rangeEnd": 75,
      "maxScore": 4.7654785 
    }
  },
  "data": [],
  "links": {
    "self": "https://de.openparliament.tv/api/v1/search?type=people&party=CDU",
    "first": "https://de.openparliament.tv/api/v1/search?type=people&party=CDU&page[number]=1&page[size]=1",
    "prev": "https://de.openparliament.tv/api/v1/search?type=people&party=CDU&page[number]=2&page[size]=1",
    "next": "https://de.openparliament.tv/api/v1/search?type=people&party=CDU&page[number]=4&page[size]=1",
    "last": "https://de.openparliament.tv/api/v1/search?type=people&party=CDU&page[number]=13&page[size]=1"
  }
}
```

## 3. Platform Statistics
`/api/v1/statistics`  



