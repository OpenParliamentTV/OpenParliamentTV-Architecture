# Open Parliament Platform Public API (DRAFT)

## 1. General Notes

The API structure is based on the [JSON:API Specification](https://jsonapi.org/format/). Whether or not we will fully implement the standard (also for PATCH requests / data updates) is to be discussed.

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
      "documentation": "https://de.openparliament.tv/api",
      "license": {
        "label": "ODC Open Database License (ODbL) v1.0",
        "link": "https://opendatacommons.org/licenses/odbl/1-0/"
      }
    },
    "requestStatus": "success" // OR "error"
  },
  "data": [], // {} OR []
  "errors": [], // EITHER "data" OR "errors"
  "links": {
    "self": "https://de.openparliament.tv/api/v1/search/media?q=Rente" // request URL
  }
}
```

**Successful** requests **MUST** include the following properties:

```yaml
{
  "meta": {
    "api": {
      "version": "1.0",
      "documentation": "https://de.openparliament.tv/api",
      "license": {
        "label": "ODC Open Database License (ODbL) v1.0",
        "link": "https://opendatacommons.org/licenses/odbl/1-0/"
      }
    },
    "requestStatus": "success"
  },
  "data": {},
  "links": {
    "self": "https://de.openparliament.tv/api/v1/search/media?q=Rente" // request URL
  }
}
```

**Errors** **MUST** include the following properties:

```yaml
{
  "meta": {
    "api": {
      "version": "1.0",
      "documentation": "https://de.openparliament.tv/api",
      "license": {
        "label": "ODC Open Database License (ODbL) v1.0",
        "link": "https://opendatacommons.org/licenses/odbl/1-0/"
      }
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
    "self": "https://de.openparliament.tv/api/v1/search/media?q=Rente" // request URL
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
  },
  "links": {
    "self": "https://de.openparliament.tv/api/v1/media/DE-198765837"
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
| parliament | min 2 chars | ?? | String |
| electoralPeriod | min 1 chars | electoralPeriod.data.attributes.number | String |
| electoralPeriodID | min 1 chars | electoralPeriod.data.id | String |
| sessionID | min 1 char | session.data.id | String |
| sessionNumber | min 1 char | session.data.attributes.number | String |
| dateFrom | date in ISO format (ex. "2017-10-28") | dateStart | String |
| dateTo | date in ISO format (ex. "2017-12-22") | dateStart | String |
| party | min 1 char | people.data.attributes.party.labelAlternative | String OR Array |
| partyID | min 1 char | organisations.data.id | String OR Array |
| faction | min 1 char | people.data.attributes.faction.labelAlternative | String OR Array |
| factionID | min 1 char | organisations.data.id | String OR Array |
| person | min 3 chars | people.data.attributes.label | String |
| personID | Wikidata ID RegEx | people.data.id | String OR Array |
| abgeordnetenwatchID | min 1 char | people.data.attributes.additionalInformation.abgeordnetenwatchID | String |
| organisationID | Wikidata ID RegEx | people.data.attributes.party.id, people.data.attributes.faction.id | String |
| context | min 3 chars | people.data.attributes.context, organisations.data.attributes.context | String |
| agendaItemID | min 2 chars | agendaItem.data.id | String |
| documentID | min 1 char | documents.data.id | String |
| termID | min 1 char | terms.data.id | String |
| id | min 4 chars | id | String |


##### `/api/v1/search/people?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| name | min 3 chars | label, firstName, lastName | String |
| type | "memberOfParliament", "unknown" | type | String |
| party | min 1 char | organisation.label, organisation.labelAlternative | String OR Array |
| partyID | Wikidata ID RegEx | partyOrganisationID | String OR Array |
| faction | min 1 char | organisation.label, organisation.labelAlternative | String OR Array |
| factionID | Wikidata ID RegEx | factionOrganisationID | String OR Array |
| organisationID | Wikidata ID RegEx | factionOrganisationID, partyOrganisationID | String |
| degree | min 1 char | degree | String |
| gender | "male", "female", "nonbinary", "bi", "queer" | gender | String |
| originID | min 1 char | originID | String |
| abgeordnetenwatchID | min 1 char | additionalInformation.abgeordnetenwatchID | String |


##### `/api/v1/search/organisations?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| name | min 3 chars | label, labelAlternative, abstract | String OR Array |
| type | min 2 char | type | String |

##### `/api/v1/search/documents?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| label | min 3 chars | label, labelAlternative, abstract | String OR Array |
| type | min 2 char | type | String |
| wikidataID | Wikidata ID RegEx | wikidataID | String |

##### `/api/v1/search/terms?`

**Parameters**  
| Parameter | Validation  | Matches | Type |
--- | --- | --- | ---
| label | min 3 chars | label, labelAlternative | String OR Array |
| type | min 2 chars | type | String |
| wikidataID | Wikidata ID RegEx | wikidataID | String |

___

#### Example Response  
```yaml
{
  "meta": {
    "api": {
      "version": "1.0",
      "documentation": "https://de.openparliament.tv/api",
      "license": {
        "label": "ODC Open Database License (ODbL) v1.0",
        "link": "https://opendatacommons.org/licenses/odbl/1-0/"
      }
    },
    "requestStatus": "success",
    "results": {
      "count": 25,
      "total": 128,
      "rangeStart": 51,
      "rangeEnd": 75,
      "maxScore": 4.7654785 /* float or null */
    }
  },
  "data": [],
  "links": {
    "self": "https://de.openparliament.tv/api/v1/search/people?party=CDU&page[number]=3&page[size]=25",
    "first": "https://de.openparliament.tv/api/v1/search/people?party=CDU&page[number]=1&page[size]=25",
    "prev": "https://de.openparliament.tv/api/v1/search/people?party=CDU&page[number]=2&page[size]=25",
    "next": "https://de.openparliament.tv/api/v1/search/people?party=CDU&page[number]=4&page[size]=25",
    "last": "https://de.openparliament.tv/api/v1/search/people?party=CDU&page[number]=13&page[size]=25"
  }
}
```

## 3. Platform Statistics
`/api/v1/statistics`  



