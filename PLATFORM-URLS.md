# Open Parliament Platform URL Structure

## 1. Search
`/search?params=`  
`/api/v1/search/ITEMTYPE?params=`  

## 2. Media, Resources & Users
### Media Details
`/media/MEDIAID`  
`/api/v1/media/MEDIAID`  

#### Media Embed
`/embed/MEDIAID`  

#### Media-specific Annotations (People, Organisations, Documents, Terms)
`/api/v1/media/MEDIAID/annotations`   

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

### User Details
`/user/USERID`  (access depends on user role / rights)
`/api/v1/user/USERID`  (access depends on user role / rights)

## 3. Login & Administration
`/login`  
`/register`   
`/registerconfirm`   
`/passwordreset`   
`/logout`  

`/dashboard` (content depends on user role / rights)  

`/manage/notifications`  

`/manage/users` (only admins)  
`/manage/users/USERID` (access depends on user role / rights)  

**Manage Data**

`/manage/data` (content depends on user role / rights)  
`/manage/data/media/MEDIAID` (access depends on user role / rights)  
`/manage/data/person/PERSONID` (access depends on user role / rights)  
`/manage/data/organisation/ORGANISATIONID` (access depends on user role / rights)  
`/manage/data/document/DOCUMENTID` (access depends on user role / rights)  
`/manage/data/term/TERMID` (access depends on user role / rights)  

`/manage/data/media/new` (access depends on user role / rights)  
`/manage/data/person/new` (access depends on user role / rights)  
`/manage/data/organisation/new` (access depends on user role / rights)  
`/manage/data/document/new` (access depends on user role / rights)  
`/manage/data/term/new` (access depends on user role / rights)  

**Administration**

`/manage/import` (only admins)  
`/manage/conflicts` (only admins)  
`/manage/conflicts/CONFLICTID` (only admins)  
`/manage/config` (only admins)  

## 4. Documentation
`/documentation`  
`/documentation/api`  

