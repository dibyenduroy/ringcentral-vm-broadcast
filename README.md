# RingCentral VM Broadcast APIs


### Overview

Voicemails are prevelant in telephony system for a long time now and and there are multiple use cases that Voicemails (VM) cater to,  both on Enterprise and Consumer domains. Here we shall mainly concentrate on the Enterprise use cases.

We at RingCentral are introducing a new set of APIs that would allow you to send Voice Mails to a bulk of receipients at the same time, through a single API call.

At a high level the features we are introducing are

1. Upload a VM to RingCentral VoiceMail library
2. Get a list of Uploaded VMs from RingCentral VoiceMail library
3. Deleting a VM message from RC VM Library
4. Broadcast VM to bulk users using a single API along with VM delivery tracking.
5. Support SMS and email notifications for broadcast voicemails


### User and App permissions

In order to use the the VM Broadcast API feature there are certain User and Application permission that the user using the App and the App itself needs to have before the feature can work.

Pre Setup and Permission Model

* Enabling the Feature for your RingCentral Account : Currenty in order to enable the feature for a RingCentral account , you need to enable the feature at an Account level.
Currently we have an Internal API to enable the VM Broadcast feature. There are also plans in future to provide it through Admin Web

This is an internal API and you need to get in touch with RingCentral Dev Support to enable the feature for your account through the API

This feature is available for  RC US/CAN (Ultimate and Enterprise) Accounts.

* User Permission Model : 

   In order to use the API, the User needs to have the following permissions

   *User Permission
      *  "Users"
      *  "VoicemailBroadcasting"
The "VoicemailBroadcasting" is a new permission model built for using this feature. This can be accessed through Service Web.By default a RingCentral Super Admin will have this permission included. It is also suggeted that developers interested in building an App with this feature can have a custom user-role created and assign it to the users who might be interested in using this App.

   
   "VoicemailBroadcasting" will be by default enabled for the Super Admin
   SuperAdmin can assign this permission to any user.

   
   *  App Permission : Voicemail


  Once the user and App permissions are setup appropriately , you should be all set to start using the API. Let us now get started with the API details


  # VM Broadcasting API Features

  
  ## Upload a VM to RingCentral VM Library : 

Before you send any Voice Mail, you would ideally like to upload some Voicemail as per your preference to the VM Library. We have an API to do just that.

  API Endpoint : `{{pas}}/restapi/v1.0/account/{{accountId}}/voicemail-library`

  Method : `POST`

  Path Parameter : accountId (required)

  Request Body Parameters (multipart/form-data, text/plain (request doesn't contain any parts with Content-Type: application/json)) : 

  * languageId : (Id of the Voicemail Language).  Can be taken for /restapi/v1.0/dictionary/language. Not a mandatory parameter.

  Parts with any names besides languageId and attachment should be ignored. If part languageId is duplicated, return error HTTP 400 CMN-406 "Duplicate value for parameter ${parameterName}: ${parameterValue} found in request."
  
	Request Body Parameters (multipart/mixed or multipart/form-data, application/json):

* languageId : (Id of the Voicemail Language).  Can be taken for /restapi/v1.0/dictionary/language. Not a mandatory parameter.

  Parts with any names besides languageId and attachment should be ignored. If part languageId is duplicated, return error HTTP 400 CMN-406 "Duplicate value for parameter ${parameterName}: ${parameterValue} found in request."

  Only one part besides attachment is allowed and its Content-Type must be application/json (error HTTP 400 MSG-391 "Unable to parse message envelope") .

 * Limitations:

    * max file size: 20Mb
	* min duration: 3 sec
	* max duration: 3 min 
	* max number of voicemails in the library: 100 (not deleted) (This number can be changed if needed)
	* supported formats: audio/mpeg, audio/mp3, audio/wav, audio/x-wav

  




  ### Request Examples 



  ```
   1. multipart/form-data,text/plain

	POST /restapi/v1.0/account/~/voicemail-library
	Content-Type: multipart/form-data; boundary=Boundary_14_2952358_1361963763144

	--Boundary_14_2952358_1361963763144
	Content-Disposition: form-data; name="languageId"
	Content-Transfer-Encoding: 8bit
	Content-Type: text/plain

	1033
	--Boundary_14_2952358_1361963763144
	Content-Disposition: form-data; name="attachment"; filename="my_custom_voicemail.mp3"
	Content-Transfer-Encoding: binary
	Content-Type: audio/mp3

<binary data>

  ```

```

 2. multipart/form-data, application/json

 	POST /restapi/v1.0/account/~/voicemail-library
	Content-Type: multipart/form-data; boundary=Boundary_14_2952358_1361963763144

	--Boundary_14_2952358_1361963763144
	Content-Type: application/json; charset=UTF-8
	Content-Transfer-Encoding: 8bit
	{
	  "language": {
	    "id": "3084"
	  }
	}

	--Boundary_14_2952358_1361963763144
	Content-Disposition: form-data; name="attachment"; filename="my_custom_voicemail.mp3"
	Content-Transfer-Encoding: binary
	Content-Type: audio/mp3

	<binary data>

```

```

3. multipart/mixed


	POST /restapi/v1.0/account/~/voicemail-library
	Content-Type: multipart/mixed; boundary=Boundary_14_2952358_1361963763144
	
	--Boundary_14_2952358_1361963763144

	Content-Type: application/json; charset=UTF-8
	Content-Transfer-Encoding: 8bit
	{
	  "language": {
	    "id": "3084"
	  }
	}
	--Boundary_14_2952358_1361963763144
	Content-Disposition: form-data; name="attachment"; filename="my_custom_voicemail.mp3"
	Content-Transfer-Encoding: binary
	Content-Type: audio/mpeg
	<binary data>


```


### Response Examples

200 OK

```

{
  "uri": "https://<RingCentralEnviornment>/restapi/v1.0/account/400770494008/voicemail-library/22008",
  "id": "22008",
  "contentType": "audio/mpeg",
  "contentUri": "https://<RingCentralEnviornment>/restapi/v1.0/account/400770494008/voicemail-library/22008/content",
  "filename": "my_custom_voicemail.mp3",
  "duration": 60,
  "language": {
    "uri": "https://<RingCentralEnviornment>/restapi/v1.0/dictionary/language/3084",
    "id": "3084",
    "name": "French (Canadian)",
    "localeCode": "fr-CA"
  }
}


```
400 Bad Request

```
{
  "errorCode": "InvalidContent",
  "message": "File with the name specified already exists",
  "errors": [
    {
      "errorCode": "MSG-399",
      "message": "File with the name specified already exists"
    }
  ]
}
```
```
{
    "errorCode": "InvalidContent",
    "message": "Voicemail duration should be [3 seconds] to [3 minutes]",
    "errors": [
        {
            "errorCode": "MSG-398",
            "message": "Voicemail duration should be [3 seconds] to [3 minutes]",
            "maxDuration": "3 minutes",
            "minDuration": "3 seconds"
        }
    ],
    "maxDuration": "3 minutes",
    "minDuration": "3 seconds"
}
```
```
{
  "errorCode": "InvalidContent",
  "message": "Attachment body is empty",
  "errors": [
    {
      "errorCode": "MSG-394",
      "message": "Attachment body is empty"
    }
  ]
}
```
```
{
  "errorCode": "InvalidContent",
  "message": "Unsupported media type for Voicemail",
  "errors": [
    {
      "errorCode": "MSG-393",
      "message": "Unsupported media type for Voicemail"
    }
  ]
}
```
Error Codes

HTTP Status Code | Error Code| Message | Reason
-----------|------------|------------|-------------
403 Forbidden|BIL-101|Feature [${parameterValue}] is not available for current service plan. parameterValue = VoicemailBroadcasting|Feature required for this function is turned off for tier by appropriate service parameter (sp = 0 - irrelevant).
403 Forbidden|BIL-103|Feature [${parameterValue}] is not available for current account. parameterValue = VoicemailBroadcasting.|Feature required for this function is turned off for account by appropriate service parameter (sp = 1 - disabled).
400 Bad Request | CMN-406 | Duplicate value for parameter [${parameterName}]: [${parameterValue}] found in request|Duplicated parameter in multipart/form-data,text/plain
400 Bad Request|MSG-391|Unable to parse message envelope|Request contains parts except json and attachment or json cannot be parsed
400 Bad Request | MSG-392 | Only one attachment is allowed|Request contains more than 1 attachments
400 Bad Request|MSG-393|Unsupported media type for Voicemail|Invalid contentType
400 Bad Request|MSG-394|Attachment body is empty|Attachment body is empty
413 Request Entity Too Large|AGW-413|Request entity too large|Attachment size limit exceeded
400 Bad Request |MSG-396|Invalid attachment content|File extension doesn't match content type or content incorrect
400 Bad Request|MSG-398|Voicemail duration should be [${minDuration}] to [${maxDuration}]|Voicemail duration is not within allowed limits
403 Forbidden|CMN-423|[${resourceName}] limit [${limit}] exceeded. resourceName = ‘Voicemails library items’|Number of Voicemails in the account’s library exceeds available limit.
400 Bad Request|MSG-370|Filename is too long, 256 characters allowed|Filename is too long
400 Bad Request | MSG-399 | File with the name specified already exists|File with the name specified and selected language already exists in the voicemails library
400 Bad Request|CMN-102|Resource for parameter [${parameterName}] is not found. parameterName = language.id|Language with the identifier specified doesn't exist

## Get VMs from your RingCentral VoiceMail Library

Given that you have already uploaded a bunch of VMs into your RingCentral library and now you want your user to pick and choose from that VM library, you will use the following API to get a list of all the VMs for your user to pick and choose before selecting and sending one.


  * API name: Get Voicemail Library Items. 
	Description: Returns list of records of the account's Voicemail Library.

	API Endpoint : {{pas}}/restapi/v1.0/account/{{accountId}}/voicemail-library

   	Method : GET 

   	Path parameters : accountId (required)

   	Name|Type|Required|Description
   	----|----|--------|-----------
   	languageId|string|no|Language internal identifier for filtering.
   	page|integer|no|Indicates the page number to retrieve. Only positive number values are allowed. Default value is '1'.
   	perPage|integer|no|Indicates the page size (number of items). Possible values: Max or a numeric value. If not specified, all records are returned on one page.


   * Request Examples

   ```
	Request: Get Voicemail Library

	GET /restapi/v1.0/account/~/voicemail-library?languageId=3084&page=1&perPage=100

   ```
	
   * Response Example

   ```
   	Response: Get Voicemail Library

   	200 OK
{
  "uri": "https://<RingCentralEnviornment>/restapi/v1.0/account/400770656008/voicemail-library?page=1&perPage=100",
  "records": [
    {
      "uri": "https://<RingCentralEnviornment>/restapi/v1.0/account/400770494008/voicemail-library/22008",
      "id": "22008",
      "contentType": "audio/mpeg",
      "contentUri": "https://<RingCentralEnviornment>/restapi/v1.0/account/400770494008/voicemail-library/22008/content",
      "filename": "my_custom_voicemail.mp3",
      "duration": 60,
      "language": {
        "uri": "https://<RingCentralEnviornment>/restapi/v1.0/dictionary/language/3084",
        "id": "3084",
        "name": "French (Canadian)",
        "localeCode": "fr-CA"
      }
    }
  ],
  "paging": {
    "page": 1,
    "totalPages": 1,
    "perPage": 100,
    "totalElements": 0
  },
  "navigation": {
    "firstPage": {
      "uri": "https://<RingCentralEnviornment>/restapi/v1.0/account/400770656008/voicemail-library/?page=1&perPage=100"
    },
    "lastPage": {
      "uri": "https://<RingCentralEnviornment>/restapi/v1.0/account/400770656008/voicemail-library/?page=1&perPage=100"
    }
  }
}

   ```	

* Get Voicemail Library Items by libraryItemId

 API name: Get Voicemail Library Item(s) by ID. 
Description: Returns individual record(s) by the given library item ID(s). Batch request is supported.


Endpoint :  GET /account/{accountId}/voicemail-library/{libraryItemId}



Method : GET 

Path parameters : accountId (required)


Name | Type | Required | Description  
------|-------|---------|-------------
languageId|string|no|Language internal identifier for filtering.
page |integer|no|Indicates the page number to retrieve. Only positive number values are allowed. Default value is '1'.
perPage|integer|no|Indicates the page size (number of items). Possible values: Max or a numeric value. If not specified, all records are returned on one page.


* Request Example

```
Request: Get Voicemail Library Item(s) by ID

GET /restapi/v1.0/account/~/voicemail-library/22008

```

* Response Example

```
	200 OK
{  
   "uri":"https://<RingCentralEnviornment>/restapi/v1.0/account/400770494008/voicemail-library/22008",
   "id":"22008",
   "contentType":"audio/mpeg",
   "contentUri":"https://<RingCentralEnviornment>/restapi/v1.0/account/400770494008/voicemail-library/22008/content",
   "filename":"my_custom_voicemail.mp3",
   "duration":60,
   "language":{  
      "uri":"https://<RingCentralEnviornment>/restapi/v1.0/dictionary/language/3084",
      "id":"3084",
      "name":"French (Canadian)",
      "localeCode":"fr-CA"
   }
}
```

HTTP Status Code | Error Code| Message | Reason
-----------|------------|------------|-------------
403 Forbidden | BIL-101|Feature [${parameterValue}] is not available for current service plan. parameterValue = VoicemailBroadcasting.|Feature required for this function is turned off for tier by appropriate service parameter (sp = 0 - irrelevant)
403 Forbidden |BIL-103|Feature [${parameterValue}] is not available for current account. parameterValue = VoicemailBroadcasting|Feature required for this function is turned off for account by appropriate service parameter (sp = 1 - disabled)
400 Bad Request |CMN-102|Resource for parameter [${parameterName}] is not found.
parameterName = libraryItemId|Library item with the identifier specified doesn't exist


Open Questions

1. Can a same VM File be uploaded in diffrent languages (I believe yes ?)


## Deleting an Item from Voicemail Library


API name: Delete Library Item(s) by ID. 
Description: Deletes record(s) by the given library item ID(s). Batch request is supported.

* It should be a soft deletion, if there are messages which refer to this library item.If library item is marked as 'deleted', when last referring message is removed, resource file and corresponding DB entry should be also deleted.*


Endpoint :  DELETE /account/{accountId}/voicemail-library/{libraryItemId}

* Request Example

```
	DELETE /account/{accountId}/voicemail-library/22008

```

* Response Example

```
	HTTP 204 No Content
```


## Download the Content of a VM from RingCentral VM Library



API name: Download the content of a VM Library item. 
Description: 

* Request Example

```
	GET {{pas}}/restapi/v1.0/account/400138306008/voicemail-library/{libraryItemId}/content
```

```
Need an example response
```

Open questions

1. Can multiple files be downloaded ?



## Send VM to Bulk Extensions


 Here we shall discuss the key feature that deals with sending VM to multiple extensions at one go.

 Sending VM to Bulk is actually an Asynchronous process, which means you submit the task and it gets executed asychronously at the backend. You can also monitor the status of the submitted task and delivery status. 

 *You cannot submit more than one VM Broadcast request concurrently at the same time, if you atemo to do so you would get a response that another job is being executed at this time*

 * Sending VM to Multiple users :


  API Name : Broadcasts voicemail from the library to multiple users

  API Endpoint : /account/{accountId}/voicemail-library/{libraryItemId}/send

  Method : POST

  Path Parameters :  accountId (required), libraryItemId (required).

  Body Parameters

  Name | Type | Required | Description
  -----|-------|--------------|-----------
 to|Collection of string|No|List of extensions’ internal identifiers for sending voicemail. Supported types: User, DigitalUser, VirtualUser, Department, SharedLinesGroup, Voicemail. Only extensions in ‘enabled’ status are allowed. If not specified, this should be treated as "all supported extensions of the account"
 from|Voicemail Sender Info|No|Sender information
 notifySipClients|boolean|No|Flag indicating that SIP notifications have to be send. Default value is false
 priority|Normal, High|No|Message priority. Default value is Normal. In case of High value message will be marked as urgent.

 Voicemail Sender Info (from)

   Name | Type | Required | Description
  -----|-------|--------------|-----------
  phoneNumber|string|No|Phone number in E.164 (with '+' sign) format. Should be a number with Caller ID feature enabled. If not specified, extension number of the logged in user will be used.
  name|string|No|Symbolic name associated with a party. 1 - 128 characters. Forbidden symbols: < > " /. If not specified, first and last name of the logged in user will be used

  ```
  Request example

  POST /restapi/v1.0/account/~/voicemail-library/5468411/send 

  Content-Type=application/json; charset=UTF-8 

{ 
   "to":[ 
      "1813741010", 
      "1289331011" 
   ], 
   "from":{ 
      "phoneNumber":"+18559100010", 
      "name":"Help desk" 
   }, 
   "notifySipClients":true, 
   "priority":"high" 
} 
  ```

Response Example

Response parameters (successful operation):

  Name | Type | Required | Description
  -----|-------|--------------|-----------
  task|TaskInfo|-|Task object

```
	HTTP 202 Accepted 

{ 
  "task" : { 
   "uri" : "https://<RingCentralEnviornment>/restapi/v1.0/account/5646451222/voicemail-library/broadcasts/402297343008-402297343008-f674e48cbb5a479bac124ca8afcffb3d", 
   "id" : "402297343008-402297343008-f674e48cbb5a479bac124ca8afcffb3d", 
    "status" : "Accepted", 
    "creationTime" : "2018-04-20T13:30:09Z", 
    "lastModifiedTime" : "2018-04-20T13:30:09Z" 
  } 
} 
```

Errors

HTTP Status Code | Error Code| Message | Reason
-----------|------------|------------|-------------
403 Forbidden|BIL-101|Feature [${parameterValue}] is not available for current service plan. parameterValue = VoicemailBroadcasting.|Feature required for this function is turned off for tier by appropriate service parameter (sp = 0 - irrelevant)
403 Forbidden|BIL-103|Feature [${parameterValue}] is not available for current account. parameterValue = VoicemailBroadcasting.|Feature required for this function is turned off for account by appropriate service parameter (sp = 1 - disabled)
403 Forbidden|MSG-400|There is another running Voicemail Broadcasting task. Try again later.|Concurrent running of several Voicemail Broadcasting tasks on one account is forbidden.
404 Not found|CMN-102|Resource for parameter [${parameterName}] is not found. parameterName = “libraryItemId”|Voicemail doesn’t exist in the account’s library
400 Not found|MSG-316|Parameter [${parameterName}] value [${parameterValue}] is invalid [Target extension not found].|Enabled extension with the identifier specified doesn't exist on the account or extension type not in supported types
400 Bad Request|CMN-101|Parameter [${parameterName}] value is invalid.|Invalid parameter is specified.
400 Bad Request|MSG-401|[${parameterValue}] cannot be used as “from” phone number|Logged in user is not allowed to use specified phone number or number of an inappropriate type.
503 Service Unavailable|CMN-201|Service Temporarily Unavailable	|Task dispatcher is Unavailable.

*Voicemail broadcasting should support sending sms and email notifications according to user configuration from “Messages & Notifications” > “Settings”.  At this phase e-mails should be composed regardless of the Voicemail to text feature*


To Check the Status of the Task you can use the following API

API Name : VM Broadcast Task Status

Endpoint : {{pas}}/restapi/v1.0/account/{{accountId}}/voicemail-library/broadcasts/{{taskId}}

Method : GET

```
  
  Request

	GET /restapi/v1.0/account/~/voicemail-library/broadcasts/402297343008-402297343008-f674e48cbb5a479bac124ca8afcffb3d 

```

Response

Name | Type | Required | Description
-----|------|----------|-------------
id|string|Yes|Task unique identifier
uri|string|No|Standard resource URI
status|Accepted,InProgress,Completed,Failed,Cancelled|No|Task actual state
creationTime|string(datetime)|No|Task creation (request) time
lastModifiedTime|string(datetime)|No|Last task status change time
details|struct|Yes|Detailed task parameters (contains only parameters specified in request
details.libraryItemId|string|No|Voicemail library item unique identifier
details.from|struct|No|Sender information
details.from.phoneNumber|string|No|Phone number
details.from.name|string|No|Symbolic name associated with a party
details.notifySipClients|boolean|No|Flag indicating that SIP notifications have to be send
details.priority|Normal, High|No|Message priority
result|struct|No|Result of task completion. Returned only for tasks in Completed state
result.errors|array|No|List of errors occurred. Returned only for tasks in Failed state
result.errors[i].errorCode|string|No|Unique logical error code
result.errors[i].message|string|No|Human-readable error message
result.processedExtensions|Array of string|No|List of successfully processed extensions (internal identifiers)
result.failedExtensions|Array of string|No|List of failed extension (internal identifiers)



```
	
	Response, Completed task

	{  
   "id":"402297343008-402297343008-f674e48cbb5a479bac124ca8afcffb3d",
   "uri":"https://<RingCentralEnviornment>/restapi/v1.0/account/5646451222/voicemail-library/broadcasts/402297343008-402297343008-f674e48cbb5a479bac124ca8afcffb3d",
   "status":"Completed",
   "creationTime":"2018-07-16T13:30:09Z",
   "lastModifiedTime":"2018-07-16T17:19:15Z",
   "details":{  
      "libraryItemId":"5468411",
      "from":{  
         "phoneNumber":"+18559100010",
         "name":"Help desk"
      },
      "notifySipClients":true,
      "priority":"high"
   },
   "result":{  
      "processedExtensions":[  
         "1813741010",
         "1289331011"
      ],
      "failedExtensions":[  
         "2563741010",
         "65454288850"
      ]
   }
}


```

```
	
	Response, Task Failed


	{  
   "id":"402297343008-402297343008-f674e48cbb5a479bac124ca8afcffb3d",
   "uri":"https://<RingCentralEnviornment>/restapi/v1.0/account/5646451222/voicemail-library/broadcasts/402297343008-402297343008-f674e48cbb5a479bac124ca8afcffb3d",
   "status":"Failed",
   "creationTime":"2018-07-16T13:30:09Z",
   "lastModifiedTime":"2018-07-16T17:19:15Z",
   "details":{  
      "libraryItemId":"5468411",
      "from":{  
         "phoneNumber":"+18559100010",
         "name":"Help desk"
      },
      "notifySipClients":true,
      "priority":"high"
   },
   "result":{  
      "errors":[  
         {  
            "errorCode":"CMN-203",
            "message":"Internal server error"
         }
      ]
   }
}
	
```










