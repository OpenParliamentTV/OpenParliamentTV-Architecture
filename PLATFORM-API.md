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

## 3. Platform Statistics
`/api/v1/statistics`  



