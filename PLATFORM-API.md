# Open Parliament Platform Public API (DRAFT)

## General Notes

For now we will roughly follow the [JSON:API Specification](https://jsonapi.org/format/). Whether or not we will fully implement the standard is to be discussed.

#### Paths
**API Documentation**  
`/api`  

**API Endpoint Base URL**  
`/api/v1`  

#### Response Objects

Response objects always include the following properties:

```yaml
{
  "meta": {},
  "data": {}, // OR
  "errors": [] 
}
```

**Successful** requests include the following meta properties:

```yaml
{
  "meta": {
    
  },
  "data": {}
}
```

**Errors** are returned like this:

```yaml
{
  "meta": {
  
  },
  "errors": [
    {
      "status": "422",   
      "code": "3",
      "source": { 
        "pointer": "/data/attributes/firstName"
      },
      "title":  "Invalid Attribute",
      "detail": "First name must contain at least three characters."
    }
  ]
}
```

#
## 1. Platform Statistics
`/api/v1/statistics`  

#
## 2. Search
`/api/v1/search?params=`  

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```


#
## 3. Media, Resources & Users
### 3.1 Media Details
`/api/v1/media/MEDIAID`  

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```

#### Media-specific Annotations (People, Organisations, Documents, Terms)
`/api/v1/media/MEDIAID/annotations`   

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```

### 3.2 Person Details
`/api/v1/person/PERSONID`  

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```

### 3.3 Organisation Details
`/api/v1/organisation/ORGANISATIONID`  

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```

### 3.4 Document Details
`/api/v1/document/DOCUMENTID`  

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```

### 3.5 Term Details
`/api/v1/term/TERMID`  

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```

### 3.6 User Details
`/api/v1/user/USERID`  

**Example Response**  
```yaml
{
  "meta": {},
  "data": {}
}
```
