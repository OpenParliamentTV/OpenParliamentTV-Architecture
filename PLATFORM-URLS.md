# Open Parliament Platform URL Structure

## 1. Search
`/search?params=`  
`/api/search?params=`  

## 2. Media, Resources & Users
### Media Details
`/media/MEDIAID?params=`  
`/api/media/MEDIAID`  

#### Media Embed
`/embed/MEDIAID`  

#### Media-specific Annotations (People, Organisations, Documents, Terms)
`/api/media/MEDIAID/annotations`   

### Person Details
`/person/PERSONID`  
`/api/person/PERSONID`  

### Organisation Details
`/organisation/ORGANISATIONID`  
`/api/organisation/ORGANISATIONID`  

### Document Details
`/document/DOCUMENTID`  
`/api/document/DOCUMENTID`  

### Term Details
`/term/TERMID`  
`/api/term/TERMID`  

### User Details
`/user/USERID`  
`/api/user/USERID`  

## 3. Login & Administration
`/login`  
`/register` ?  
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
