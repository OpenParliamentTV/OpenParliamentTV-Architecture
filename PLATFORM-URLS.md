# Open Parliament Platform URL Structure

## 1. Search
`/search?params=`  
`/api/v1/search/ITEMTYPE?params=`  
`/api/v1/autocomplete/ITEMTYPE?params=`  

## 2. Media & Resources
### Media Details
`/media/MEDIAID`  
`/api/v1/media/MEDIAID`  

#### Embed
`/embed/entity?type=TYPE&id=ID`  

### Person Details
`/person/PERSONID`  
`/api/v1/person/PERSONID`  

### Organisation Details
`/organisation/ORGANISATIONID`  
`/api/v1/organisation/ORGANISATIONID`  

### Document Details
`/document/DOCUMENTID`  
`/api/v1/document/DOCUMENTID`  

### Term Details
`/term/TERMID`  
`/api/v1/term/TERMID`  

### Agenda Item Details
`/agendaItem/AGENDAITEMID`  
`/api/v1/agendaItem/AGENDAITEMID`  

### Session Details
`/session/SESSIONID`  
`/api/v1/session/SESSIONID`  

### Electoral Period Details
`/electoralPeriod/ELECTORALPERIODID`  
`/api/v1/electoralPeriod/ELECTORALPERIODID`  

## 3. Login & Administration
`/login`  
`/register`  
`/registerConfirm`  
`/password-reset`  
`/logout`  

`/manage` (content depends on user role / rights)  

`/manage/notifications`  
`/manage/settings` (only admins)  

`/manage/users` (only admins)  
`/manage/users/USERID` (access depends on user role / rights)  

**Manage Media**

`/manage/media` (access depends on user role / rights)  
`/manage/media/MEDIAID` (only admins)  
`/manage/media/new` (only admins)  

**Manage Entities**

`/manage/entities` (access depends on user role / rights)  
`/manage/entity-suggestions` (access depends on user role / rights)  

**Manage Structure**

`/manage/structure` (only admins)  
`/manage/structure/electoralPeriod/ELECTORALPERIODID` (only admins)  
`/manage/structure/session/SESSIONID` (only admins)  
`/manage/structure/agendaItem/AGENDAITEMID` (only admins)  

**Administration**

`/manage/import` (only admins)  
`/manage/conflicts` (only admins)  
`/manage/conflicts/CONFLICTID` (only admins)  

## 4. Other Pages
`/about`  
`/datapolicy`  
`/imprint`  
`/press`  
`/statistics`  
