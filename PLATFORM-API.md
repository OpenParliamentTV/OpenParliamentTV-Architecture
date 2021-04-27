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

```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```

#
## 1. Platform Statistics
`/api/v1/statistics`  

#
## 2. Search
`/api/v1/search?params=`  

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```


#
## 3. Media, Resources & Users
### 3.1 Media Details
`/api/v1/media/MEDIAID`  

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```

#### Media-specific Annotations (People, Organisations, Documents, Terms)
`/api/v1/media/MEDIAID/annotations`   

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```

### 3.2 Person Details
`/api/v1/person/PERSONID`  

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```

### 3.3 Organisation Details
`/api/v1/organisation/ORGANISATIONID`  

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```

### 3.4 Document Details
`/api/v1/document/DOCUMENTID`  

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```

### 3.5 Term Details
`/api/v1/term/TERMID`  

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```

### 3.6 User Details
`/api/v1/user/USERID`  

**Example Response**  
```json
{
	"meta": {},
	"data": {}, // OR
	"errors": {} 
}
```
