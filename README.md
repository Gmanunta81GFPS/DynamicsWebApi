# DynamicsWebApi for Microsoft Dynamics 365 CE (CRM) / Common Data Service Web API

[![Travis](https://img.shields.io/travis/AleksandrRogov/DynamicsWebApi.svg?style=flat-square)](https://travis-ci.org/AleksandrRogov/DynamicsWebApi)
[![Coveralls](https://img.shields.io/coveralls/AleksandrRogov/DynamicsWebApi.svg?style=flat-square)](https://coveralls.io/github/AleksandrRogov/DynamicsWebApi)
![npm](https://img.shields.io/npm/dm/dynamics-web-api?style=flat-square)
![npm](https://img.shields.io/npm/dt/dynamics-web-api?style=flat-square)

DynamicsWebApi is a Microsoft Dynamics 365 CE (CRM) / Common Data Service Web API helper library written in JavaScript.
It is compatible with: Common Data Service, Dynamics 365 CE (online), Dynamics 365 CE (on-premises), Dynamics CRM 2016, Dynamics CRM Online

Please check [DynamicsWebApi Wiki](../../wiki/) where you will find documentation to DynamicsWebApi API and more.

Libraries for browsers can be found in [dist](/dist/) folder.

If you feel that this project saved your time and you would like to support it, then please feel free to donate: [![PayPal.Me](/extra/paypal.png)](https://paypal.me/alexrogov)

Please check [suggestions and contributions](#contributions) section to learn more on how you can help to improve this project.

**DynamicsWebApi v2 is coming!**
There will be breaking changes between v1 and v2. One of them: v2 will not have simple requests, only the advanced ones, therefore I highly recommend using request objects for making requests.
Also v2 will be written in TypeScript and include numerous optimizations. I am very excited to finally release the new version and I hope you too! Stay tuned!

**Important!** For some reason, npm was not removing `.git` folder from a published package, 
even though [it should have done it by default](https://docs.npmjs.com/misc/developers#keeping-files-out-of-your-package), therefore
`npm update dynamics-web-api` was not working properly. If you see an error during an update of the package, 
please go to `node_modules\dynamics-web-api` of your application and remove `.git` directory manually. This error has been fixed in `v.1.4.7`.

Please note, that "Dynamics 365" in this readme refers to Microsoft Dynamics 365 Customer Engagement / Common Data Service.

## Table of Contents

* [Getting Started](#getting-started)
  * [DynamicsWebApi as a Dynamics 365 web resource](#dynamicswebapi-as-a-dynamics-365-web-resource)
  * [DynamicsWebApi for Node.js](#dynamicswebapi-for-nodejs)
  * [Configuration](#configuration)
    * [Configuration Parameters](#configuration-parameters)
* [Request Examples](#request-examples)
  * [Create a record](#create-a-record)
  * [Update a record](#update-a-record)
  * [Update a single property value](#update-a-single-property-value)
  * [Upsert a record](#upsert-a-record)
  * [Delete a record](#delete-a-record)
    * [Delete a single property value](#delete-a-single-property-value)
  * [Retrieve a record](#retrieve-a-record)
  * [Retrieve multiple records](#retrieve-multiple-records)
    * [Change Tracking](#change-tracking)
    * [Retrieve All records](#retrieve-all-records)
  * [Count](#count)
    * [Count limitation workaround](#count-limitation-workaround)
  * [Associate](#associate)
  * [Associate for a single-valued navigation property](#associate-for-a-single-valued-navigation-property)
  * [Disassociate](#disassociate)
  * [Disassociate for a single-valued navigation property](#disassociate-for-a-single-valued-navigation-property)
  * [Fetch XML Request](#fetch-xml-request)
    * [Fetch All records](#fetch-all-records)
  * [Execute Web API functions](#execute-web-api-functions)
  * [Execute Web API actions](#execute-web-api-actions)
  * [Execute Batch Operations](#execute-batch-operations)
  * [Working with Metadata Definitions](#working-with-metadata-definitions)
    * [Create Entity](#create-entity)
    * [Retrieve Entity](#retrieve-entity)
    * [Update Entity](#update-entity)
    * [Retrieve Multiple Entities](#retrieve-multiple-entities)
    * [Create Attribute](#create-attribute)
    * [Retrieve Attribute](#retrieve-attribute)
    * [Update Attribute](#update-attribute)
    * [Retrieve Multiple Attributes](#retrieve-multiple-attributes)
    * [Use requests to query Entity and Attribute metadata](#use-requests-to-query-entity-and-attribute-metadata)
	* [Create Relationship](#create-relationship)
	* [Update Relationship](#update-relationship)
	* [Delete Relationship](#delete-relationship)
	* [Retrieve Relationship](#retrieve-relationship)
	* [Retrieve Multiple Relationships](#retrieve-multiple-relationships)
	* [Create Global Option Set](#create-global-option-set)
	* [Update Global Option Set](#update-global-option-set)
	* [Delete Global Option Set](#delete-global-option-set)
	* [Retrieve Global Option Set](#retrieve-global-option-set)
	* [Retrieve Multiple Global Option Sets](#retrieve-multiple-global-option-sets)
* [Formatted Values and Lookup Properties](#formatted-values-and-lookup-properties)
* [Using Alternate Keys](#using-alternate-keys)
* [Making requests using Entity Logical Names](#making-requests-using-entity-logical-names)
* [Using TypeScript Declaration Files](#using-typescript-declaration-files)
* [JavaScript Promises](#javascript-promises)
* [JavaScript Callbacks](#javascript-callbacks)
* [Contributions](#contributions)

## Getting Started

### DynamicsWebApi as a Dynamics 365 web resource
In order to use DynamicsWebApi inside Dynamics 365 you need to download a browser version of the library, it can be found in [dist](/dist/) folder.

Upload a script as a JavaScript Web Resource, place on the entity form or refer to it in your HTML Web Resource and then initialize the main object:

```js
//DynamicsWebApi makes calls to Web API v9.1 if a configuration not set
const dynamicsWebApi = new DynamicsWebApi();

let response = await dynamicsWebApi.executeUnboundFunction("WhoAmI");
Xrm.Navigation.openAlertDialog({ text: `Hello Dynamics 365! My id is: ${response.UserId}` });
```

### DynamicsWebApi for Node.js
DynamicsWebApi can be used as Node.js module to access Dynamics 365 Web API using OAuth.

First of all, install a package from NPM:

```shell
npm install dynamics-web-api --save
```

Then include it in your file:

```js
var DynamicsWebApi = require('dynamics-web-api');
```

At this moment DynamicsWebApi does not fetch authorization tokens, so you will need to acquire OAuth token in your code and pass it to the DynamicsWebApi.
Token can be aquired using [ADAL for Node.js](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs) or you can write your own functionality, as it is described [here](http://alexanderdevelopment.net/post/2016/11/23/dynamics-365-and-node-js-integration-using-the-web-api/).

Here is a sample using `adal-node`:

```js
var DynamicsWebApi = require('dynamics-web-api');
var AuthenticationContext = require('adal-node').AuthenticationContext;

//the following settings should be taken from Azure for your application
//and stored in app settings file or in global variables

//OAuth Token Endpoint
var authorityUrl = 'https://login.microsoftonline.com/00000000-0000-0000-0000-000000000011/oauth2/token';
//CRM Organization URL
var resource = 'https://myorg.crm.dynamics.com';
//Dynamics 365 Client Id when registered in Azure
var clientId = '00000000-0000-0000-0000-000000000001';
var username = 'crm-user-name';
var password = 'crm-user-password';

var adalContext = new AuthenticationContext(authorityUrl);

//add a callback as a parameter for your function
function acquireToken(dynamicsWebApiCallback){
    //a callback for adal-node
    function adalCallback(error, token) {
        if (!error){
            //call DynamicsWebApi callback only when a token has been retrieved
            dynamicsWebApiCallback(token);
        }
        else{
            console.log('Token has not been retrieved. Error: ' + error.stack);
        }
    }

    //call a necessary function in adal-node object to get a token
    adalContext.acquireTokenWithUsernamePassword(resource, username, password, clientId, adalCallback);
}

//create DynamicsWebApi object
var dynamicsWebApi = new DynamicsWebApi({
    webApiUrl: 'https://myorg.api.crm.dynamics.com/api/data/v9.1/',
    onTokenRefresh: acquireToken
});

//call any function
dynamicsWebApi.executeUnboundFunction("WhoAmI").then(function (response) {
    console.log('Hello Dynamics 365! My id is: ' + response.UserId);
}).catch(function(error){
    console.log(error.message);
});
```

### Configuration
To initialize a new instance of DynamicsWebApi with a configuration object, please use the following code:

#### Web browser

```js
var dynamicsWebApi = new DynamicsWebApi({ webApiVersion: '9.1' });
```

#### Node.js

```js
var dynamicsWebApi = new DynamicsWebApi({
    webApiUrl: 'https://myorg.api.crm.dynamics.com/api/data/v9.1/',
    onTokenRefresh: acquireToken
});
```

You can set a configuration dynamically if needed:

```js
//or can be set dynamically
dynamicsWebApi.setConfig({ webApiVersion: '9.1' });
```

#### Configuration Parameters
Property Name | Type | Description
------------ | ------------- | -------------
impersonate | String | Impersonates a user based on their systemuserid by adding a "MSCRMCallerID" header. A String representing the GUID value for the Dynamics 365 systemuserid. [More Info](https://docs.microsoft.com/en-us/powerapps/developer/common-data-service/webapi/impersonate-another-user-web-api)
impersonateAAD | String | Impersonates a user based on their Azure Active Directory (AAD) object id by passing that value along with the header "CallerObjectId". A String should represent a GUID value. [More Info](https://docs.microsoft.com/en-us/powerapps/developer/common-data-service/webapi/impersonate-another-user-web-api)
includeAnnotations | String | Defaults Prefer header with value "odata.include-annotations=" and the specified annotation. Annotations provide additional information about lookups, options sets and other complex attribute types.
maxPageSize | Number | Defaults the odata.maxpagesize preference. Use to set the number of entities returned in the response.
onTokenRefresh | Function | A callback function that triggered when DynamicsWebApi requests a new OAuth token. (At this moment it is done before each call to Dynamics 365, as [recommended by Microsoft](https://msdn.microsoft.com/en-ca/library/gg327838.aspx#Anchor_2)).
returnRepresentation | Boolean | Defaults Prefer header with value "return=representation". Use this property to return just created or updated entity in a single request.
timeout | Number | Sets a number of milliseconds before a request times out.
useEntityNames | Boolean | Indicates whether to use entity logical names instead of collection logical names during requests.
webApiUrl | String | A complete URL string to Web API. Example of the URL: "https://myorg.api.crm.dynamics.com/api/data/v9.1/". If it is specified then webApiVersion property will not be used even if it is not empty. 
webApiVersion | String | Version of the Web API. Default version is "9.1".

Configuration property `webApiVersion` is required only when DynamicsWebApi used inside CRM. 
Property `webApiUrl` is required when DynamicsWebApi used externally.

**Important!** If both configuration properties set then `webApiUrl` will have a higher priority than `webApiVersion`, so the last one will be skipped.

**Important!** Please note, if you are using `DynamicsWebApi` **outside Microsoft Dynamics 365** and set `useEntityNames` to `true` **the first request** to Web Api 
will fetch `LogicalCollectionName` and `LogicalName` from entity metadata for all entities. It does not happen when `DynamicsWebApi`
is used in Microsoft Dynamics 365 Web Resources (there is no additional request, no impact on perfomance).

## Request Examples

For the object reference please use [DynamicsWebApi Wiki](../../wiki/).

The following table describes all __possible__ properties that can be set for `request` object. __Please note,__ not all operaions accept all properties and if 
by mistake an invalid property has been specified you will receive either an error saying that the request is invalid or the response will not have expected results.

Property Name | Type | Operation(s) Supported | Description
------------ | ------------- | ------------- | -------------
action | Object | `executeUnboundAction` | A JavaScript object that represents a Dynamics 365 Web API action.
actionName | String | `executeUnboundAction` | Web API Action name.
apply | String | `retrieveMultiple`, `retrieveAll` | Sets the $apply system query option to aggregate and group your data dynamically. [More Info](https://docs.microsoft.com/en-us/powerapps/developer/common-data-service/webapi/query-data-web-api#aggregate-and-grouping-results)
async | Boolean | All | **Important! XHR requests only!** Indicates whether the requests should be made synchronously or asynchronously. Default value is `true` (asynchronously).
collection | String | All | Entity Collection name.
contentId | String | `create`, `update`, `upsert`, `deleteRecord` | **BATCH REQUESTS ONLY!** Sets Content-ID header or references request in a Change Set. [More Info](https://www.odata.org/documentation/odata-version-3-0/batch-processing/)
count | Boolean | `retrieveMultiple`, `retrieveAll` | Boolean that sets the $count system query option with a value of true to include a count of entities that match the filter criteria up to 5000 (per page). Do not use $top with $count!
data | Object | `create`, `update`, `upsert` | A JavaScript object that represents Dynamics 365 entity, action, metadata and etc. 
duplicateDetection | Boolean | `create`, `update`, `upsert` | **Web API v9+ only!** Boolean that enables duplicate detection. [More Info](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/webapi/update-delete-entities-using-web-api#check-for-duplicate-records)
expand | Array | `retrieve`, `retrieveMultiple`, `create`, `update`, `upsert` | An array of Expand Objects (described below the table) representing the $expand OData System Query Option value to control which related records are also returned.
fetchXml | String | `fetch`, `fetchAll` | Property that sets FetchXML - a proprietary query language that provides capabilities to perform aggregation.
filter | String | `retrieve`, `retrieveMultiple`, `retrieveAll` | Use the $filter system query option to set criteria for which entities will be returned.
ifmatch | String | `retrieve`, `update`, `upsert`, `deleteRecord` | Sets If-Match header value that enables to use conditional retrieval or optimistic concurrency in applicable requests. [More Info](https://msdn.microsoft.com/en-us/library/mt607711.aspx)
ifnonematch | String | `retrieve`, `upsert` | Sets If-None-Match header value that enables to use conditional retrieval in applicable requests. [More Info](https://msdn.microsoft.com/en-us/library/mt607711.aspx).
impersonate | String | All | Impersonates a user based on their systemuserid by adding a "MSCRMCallerID" header. A String representing the GUID value for the Dynamics 365 systemuserid. [More Info](https://docs.microsoft.com/en-us/powerapps/developer/common-data-service/webapi/impersonate-another-user-web-api)
impersonateAAD | String | All | Impersonates a user based on their Azure Active Directory (AAD) object id by passing that value along with the header "CallerObjectId". A String should represent a GUID value. [More Info](https://docs.microsoft.com/en-us/powerapps/developer/common-data-service/webapi/impersonate-another-user-web-api)
includeAnnotations | String | `retrieve`, `retrieveMultiple`, `retrieveAll`, `create`, `update`, `upsert` | Sets Prefer header with value "odata.include-annotations=" and the specified annotation. Annotations provide additional information about lookups, options sets and other complex attribute types.
key | String | `retrieve`, `create`, `update`, `upsert`, `deleteRecord` | A String representing collection record's Primary Key (GUID) or Alternate Key(s).
maxPageSize | Number | `retrieveMultiple`, `retrieveAll` | Sets the odata.maxpagesize preference value to request the number of entities returned in the response.
mergeLabels | Boolean | `update` | **Metadata Update only!** Sets `MSCRM.MergeLabels` header that controls whether to overwrite the existing labels or merge your new label with any existing language labels. Default value is `false`. [More Info](https://msdn.microsoft.com/en-us/library/mt593078.aspx#bkmk_updateEntities)
metadataAttributeType | String | `retrieve`, `update` | Casts the Attributes to a specific type. (Used in requests to Attribute Metadata) [More Info](https://msdn.microsoft.com/en-us/library/mt607522.aspx#Anchor_4)
navigationProperty | String | `retrieve`, `create`, `update` | A String representing the name of a single-valued navigation property. Useful when needed to retrieve information about a related record in a single request.
navigationPropertyKey | String | `retrieve`, `create`, `update` | A String representing navigation property's Primary Key (GUID) or Alternate Key(s). (For example, to retrieve Attribute Metadata)
noCache | Boolean | All | If set to `true`, DynamicsWebApi adds a request header `Cache-Control: no-cache`. Default value is `false`.
orderBy | Array | `retrieveMultiple`, `retrieveAll` | An Array (of Strings) representing the order in which items are returned using the $orderby system query option. Use the asc or desc suffix to specify ascending or descending order respectively. The default is ascending if the suffix isn't applied.
pageNumber | Number | `fetch` | Sets a page number for Fetch XML request ONLY!
pagingCookie | String | `fetch` | Sets a paging cookie for Fetch XML request ONLY!
returnRepresentation | Boolean | `create`, `update`, `upsert` | Sets Prefer header request with value "return=representation". Use this property to return just created or updated entity in a single request.
savedQuery | String | `retrieve` | A String representing the GUID value of the saved query.
select | Array | `retrieve`, `retrieveMultiple`, `retrieveAll`, `update`, `upsert` | An Array (of Strings) representing the $select OData System Query Option to control which attributes will be returned.
timeout | Number | All | Sets a number of milliseconds before a request times out.
token | String | All | Authorization Token. If set, onTokenRefresh will not be called.
top | Number | `retrieveMultiple`, `retrieveAll` | Limit the number of results returned by using the $top system query option. Do not use $top with $count!
trackChanges | Boolean | `retrieveMultiple`, `retrieveAll` | Sets Prefer header with value 'odata.track-changes' to request that a delta link be returned which can subsequently be used to retrieve entity changes. __Important!__ Change Tracking must be enabled for the entity. [More Info](https://docs.microsoft.com/en-us/powerapps/developer/common-data-service/use-change-tracking-synchronize-data-external-systems#enable-change-tracking-for-an-entity)
userQuery | String | `retrieve` | A String representing the GUID value of the user query.

The following table describes Expand Object properties:

Property Name | Type | Description
------------ | ------------- | -------------
expand | Array | An array of Expand Objects representing the $expand OData System Query Option value to control which related records are also returned.
filter | String | Use the $filter system query option to set criteria for which related entities will be returned.
orderBy | Array | An Array (of Strings) representing the order in which related items are returned using the $orderby system query option. Use the asc or desc suffix to specify ascending or descending order respectively. The default is ascending if the suffix isn't applied.
property | String | A name of a single-valued navigation property which needs to be expanded.
select | Array | An Array (of Strings) representing the $select OData System Query Option to control which attributes will be returned.
top | Number | Limit the number of results returned by using the $top system query option.

All requests to Web API that have long URLs (more than 2000 characters) are automatically converted to a Batch Request.
This feature is very convenient when you make a call with big Fetch XMLs. No special parameters needed to do a convertation.

### Create a record

#### TypeScript

```ts
//declaring interface for a Lead entity (declaration can be done in d.ts file)
interface Lead {
    leadid?: string,
    subject?: string,
    firstname?: string,
    lastname?: string,
    jobtitle?: string
}

//init an object representing Dynamics 365 entity
let lead: Lead = {
    subject: "Test WebAPI",
    firstname: "Test",
    lastname: "WebAPI",
    jobtitle: "Title"
};

//init DynamicsWebApi request
let request: DynamicsWebApi.CreateRequest = {
    collection: "leads",
    data: lead,
    returnRepresentation: true
}

//call dynamicsWebApi.create function
//let's use Lead here because we set returnRepresentation to `true`
//if the type was ommitted here then the result would be of type `any`
let result = await dynamicsWebApi.create<Lead>(request);

//do something with a record here
let leadId = result.leadid;
```

#### JavaScript

```js
//init an object representing Dynamics 365 entity
let lead = {
    subject: "Test WebAPI",
    firstname: "Test",
    lastname: "WebAPI",
    jobtitle: "Title"
};

//init DynamicsWebApi request
let request = {
    collection: "leads",
    data: lead,
    returnRepresentation: true
}

//call dynamicsWebApi.create function
let result = await dynamicsWebApi.create(request);

//do something with a record here
let subject = result.subject;
```

### Update a record

#### TypeScript

```ts
//declaring interface for a Lead entity (declaration can be done in d.ts file)
interface Lead {
    leadid?: string,
    subject?: string,
    fullname?: string,
    jobtitle?: string
}

//init DynamicsWebApi request
let request: DynamicsWebApi.UpdateRequest = {
    key: "7d577253-3ef0-4a0a-bb7f-8335c2596e70",
    collection: "leads",
    data: {
        subject: "Test update",
        jobtitle: "Developer"
    },
    returnRepresentation: true,
    select: ["fullname"]
};

//call dynamicsWebApi.update function
//let's use Lead here because we set returnRepresentation to `true`
//if the type was ommitted here then the result would be of type `any`
let result = await dynamicsWebApi.update<Lead>(request);

//do something with a fullname of a recently updated entity record
let fullname = result.fullname;
```

#### JavaScript

```js
//init DynamicsWebApi request
let request = {
    key: '7d577253-3ef0-4a0a-bb7f-8335c2596e70',
    collection: "leads",
    data: {
        subject: "Test update",
        jobtitle: "Developer"
    },
    returnRepresentation: true,
    select: ["fullname"]
};

let result = await dynamicsWebApi.update(request);

//do something with a fullname of a recently updated entity record
let fullname = result.fullname;
```

### Update a single property value

#### TypeScript

```ts
//initialize key value pair object
let request: DynamicsWebApi.UpdateSinglePropertyRequest = {
    collection: "leads",
    key: "7d577253-3ef0-4a0a-bb7f-8335c2596e70",
    fieldValuePair: { subject: "Update Single" }
};

//perform an update single property operation
await dynamicsWebApi.updateSingleProperty(request);

//do something after a succesful operation
```

#### JavaScript

```js
//initialize key value pair object
let request = {
    collection: "leads",
    key: "7d577253-3ef0-4a0a-bb7f-8335c2596e70",
    fieldValuePair: { subject: "Update Single" }
};

//perform an update single property operation
await dynamicsWebApi.updateSingleProperty(request);

//do something after a succesful operation
```

### Upsert a record

#### TypeScript

```ts
const leadId = "7d577253-3ef0-4a0a-bb7f-8335c2596e70";

let request: DynamicsWebApi.UpsertRequest = {
    key: leadId,
    collection: "leads",
    returnRepresentation: true,
    select: ["fullname"],
    data: {
        subject: "Test upsert"
    },
    ifnonematch: "*" //to prevent update
};

let result = dynamicsWebApi.upsert<Lead>(request);
if (result != null) {
    //record created
}
else {
    //update prevented
}
```


#### JavaScript

```js
const leadId = "7d577253-3ef0-4a0a-bb7f-8335c2596e70";

let request = {
    key: leadId,
    collection: "leads",
    returnRepresentation: true,
    select: ["fullname"],
    data: {
        subject: "Test upsert"
    },
    ifnonematch: "*" //to prevent update
};

let result = dynamicsWebApi.upsert(request);
if (result != null) {
    //record created
}
else {
    //update prevented
}
```

### Delete a record

#### TypeScript

```ts
//delete with optimistic concurrency
let request: DynamicsWebApi.DeleteRequest = {
    key: recordId,
    collection: "leads",
    ifmatch: 'W/"470867"'
}

let isDeleted = await dynamicsWebApi.deleteRecord(request);
if (isDeleted){
    //record has been deleted
}
else{
    //record has not been deleted
}
```

#### JavaScript

```js
//delete with optimistic concurrency
let request = {
    key: recordId,
    collection: "leads",
    ifmatch: 'W/"470867"'
}

let isDeleted = await dynamicsWebApi.deleteRecord(request);
if (isDeleted){
    //record has been deleted
}
else{
    //record has not been deleted
}
```
### Retrieve a record

#### TypeScript

```ts
//declaring interface for a Lead entity (declaration can be done in d.ts file)
interface Lead {
    leadid?: string,
    subject?: string,
    fullname?: string,
    jobtitle?: string
}

let request: DynamicsWebApi.RetrieveRequest = {
    key: "7d577253-3ef0-4a0a-bb7f-8335c2596e70",
    collection: "leads",
    select: ["fullname", "subject"],

    //ETag value with the If-None-Match header to request data to be retrieved only 
    //if it has changed since the last time it was retrieved.
    ifnonematch: 'W/"468026"',

    //Retrieved record will contain formatted values
    includeAnnotations: "OData.Community.Display.V1.FormattedValue"
};

//call dynamicsWebApi.retrieve function
//if the Lead type was ommitted here then the result would be of type `any`
let result = await dynamicsWebApi.retrieve<Lead>(request);

//do something with a retrieved record
```

#### JavaScript

```js
let request = {
    key: "7d577253-3ef0-4a0a-bb7f-8335c2596e70",
    collection: "leads",
    select: ["fullname", "subject"],

    //ETag value with the If-None-Match header to request data to be retrieved only 
    //if it has changed since the last time it was retrieved.
    ifnonematch: 'W/"468026"',

    //Retrieved record will contain formatted values
    includeAnnotations: "OData.Community.Display.V1.FormattedValue"
};

//do something with a retrieved record
let result = await dynamicsWebApi.retrieve(request);
```

#### Retrieve a reference to related record using a single-valued navigation property

It is possible to retrieve a reference to the related entity. In order to do that: `select` property
must contain only a single value representing a name of a [single-valued navigation property](https://msdn.microsoft.com/en-us/library/mt607990.aspx#Anchor_5) 
and it must have a suffix `/$ref` attached to it. Example:

```ts
const leadId = "7d577253-3ef0-4a0a-bb7f-8335c2596e70";

let request: DynamicsWebApi.RetrieveRequest = {
    collection: "leads",
    key: leadId,
    select: ["onwerid/$ref"]
}

//perform a retrieve operaion
let reference = await dynamicsWebApi.retrieve(request);
let ownerId = reference.id;
let collectionName = reference.collection; // systemusers or teams
```

#### Retrieve a related record data using a single-valued navigation property

In order to retrieve a related record by a single-valued navigation property you need to add a prefix "/" to the __first__ element in a `select` array: 
`select: ["/ownerid", "fullname"]`. The first element must be the name of a [single-valued navigation property](https://msdn.microsoft.com/en-us/library/mt607990.aspx#Anchor_5) 
and it must contain a prefix "/"; all other elements in a `select` array will represent attributes of __the related entity__. Examples:

```ts
const recordId = "7d577253-3ef0-4a0a-bb7f-8335c2596e70";

let request: DynamicsWebApi.RetrieveRequest = {
    key: recordId,
    collection: "new_tests",
    select: ["/new_ParentLead", "fullname", "subject"]
}

//perform a retrieve operaion
let parentLead = await dynamicsWebApi.retrieve<Lead>(request);

let fullname = parentLead.fullname;
//... and etc
```

Same result can be achieved by setting `request.navigationProperty`:

```ts
var request: DynamicsWebApi.RetrieveRequest = {
    key: recordId,
    collection: "new_tests",
    navigationProperty: "new_ParentLead", //use request.navigationProperty
    select: ["fullname", "subject"]
}

//perform a retrieve operaion
let parentLead = await dynamicsWebApi.retrieve<Lead>(request);

let fullname = parentLead.fullname;
//... and etc
```

### Retrieve multiple records

#### TypeScript

```ts
//declaring interface for a Lead entity (declaration can be done in d.ts file)
interface Lead {
    leadid?: string,
    subject?: string,
    fullname?: string
}

let request: DynamicsWebApi.RetrieveMultipleRequest = {
    collection: "leads",
    select: ["fullname", "subject"],
    filter: "statecode eq 0",
    maxPageSize: 5,
    count: true
}

let response = await dynamicsWebApi.retrieveMultiple<Lead>(request);

let count = response.oDataCount;
let nextLink = response.oDataNextLink;
let records = response.value;
```

#### JavaScript

```js
let request = {
    collection: "leads",
    select: ["fullname", "subject"],
    filter: "statecode eq 0",
    maxPageSize: 5,
    count: true
}

let result = await dynamicsWebApi.retrieveMultiple(request);

let count = response.oDataCount;
let nextLink = response.oDataNextLink;
let records = response.value;
//do something else with a records array. Access a record: records[0].subject;
```

#### Change Tracking

```ts
//set the request parameters
var request: DynamicsWebApi.RetrieveMultipleRequest = {
    collection: "leads",
    select: ["fullname", "subject"],
    trackChanges: true
};

//perform a multiple records retrieve operation (1)
let response1 = await dynamicsWebApi.retrieveMultiple<Lead>(request).then(function (response) {

let deltaLink = response1.oDataDeltaLink;
//make other requests to Web API
//...
//(2) only retrieve changes:
let response2 = await dynamicsWebApi.retrieveMultiple<Lead>(request, deltaLink);
//response2 contains only changed records between the first retrieveMultiple (1) and the second one (2)
```

#### Retrieve All records

The following function retrieves records and goes through all pages automatically.

```ts
//perform a multiple records retrieve operation
let response = await dynamicsWebApi.retrieveAll<Lead>({ 
    collection: "leads", 
    select: ["fullname", "subject"], 
    filter: "statecode eq 0"
});

let records = response.value;
//do something else with a records array. Access a record: records[0].subject;
```

### Count

It is possible to count records separately from RetrieveMultiple call. In order to do that use the following snippet:

IMPORTANT! The count value does not represent the total number of entities in the system. It is limited by the maximum number of entities that can be returned.

```ts
let count = await dynamicsWebApi.count({ 
    collection: "leads", 
    filter: "statecode eq 0"
});

//do something with count here
```

#### Count limitation workaround

The following function can be used to count all records in a collection. It's a workaround and just counts the number of objects in the array 
returned in `retrieveAll`.

Downside of this workaround is that it does not only return a count number but also all data for records in a collection. In order to make a small
optimisation always provide a column for select paramete, it will reduce a size of the response significantly. 

```ts
let count = await dynamicsWebApi.countAll({ 
    collection: "leads", 
    filter: "statecode eq 0",
    //if you use this workaround, always provide a column 
    //to limit a response size
    select: ["leadid"]
});
```

FYI, in the majority of cases it is better to use Fetch XML aggregation, but take into a consideration that it is also limited to 50000 records 
by default.

### Associate

```ts
let accountId = "00000000-0000-0000-0000-000000000001";
let leadId = "00000000-0000-0000-0000-000000000002";

//associate lead reacord to account
let request: DynamicsWebApi.AssociateRequest = {
    collection: "accounts",
    primaryKey: accountId,
    relationshipName: "lead_parent_account",
    relatedCollection: "leads",
    relatedKey: leadId
}

await dynamicsWebApi.associate(request);
//does not have any return value
```

JavaScript sample is the same only without a request type declaration.

### Associate for a single-valued navigation property

The name of a single-valued navigation property can be retrieved by using a `GET` request with a header `Prefer:odata.include-annotations=Microsoft.Dynamics.CRM.associatednavigationproperty`, 
then individual records in the response will contain the property `@Microsoft.Dynamics.CRM.associatednavigationproperty` which is the name of the needed navigation property.

For example, there is an entity with a logical name `new_test`, it has a lookup attribute to `lead` entity called `new_parentlead` and schema name `new_ParentLead` which is needed single-valued navigation property.

```ts
let new_testid = "00000000-0000-0000-0000-000000000001";
let leadId = "00000000-0000-0000-0000-000000000002";

let request: DynamicsWebApi.AssociateSingleValuedRequest = {
    collection: "new_tests",
    primaryKey: new_testid,
    navigationProperty: "new_ParentLead",
    relatedCollection: "leads",
    relatedKey: leadId
}

await dynamicsWebApi.associateSingleValued(request);
//does not have any return value
```

JavaScript sample is the same only without a request type declaration.

### Disassociate

```ts
let accountId = "00000000-0000-0000-0000-000000000001";
let leadId = "00000000-0000-0000-0000-000000000002";

let request: DynamicsWebApi.DisassociateRequest = {
    collection: "accounts",
    primaryKey: accountId,
    relationshipName: "lead_parent_account",
    relatedKey: leadId
}

await dynamicsWebApi.disassociate(request);
//does not have any return value
```

JavaScript sample is the same only without a request type declaration.

### Disassociate for a single-valued navigation property
Current request removes a reference to an entity for a single-valued navigation property. 
The following code snippet uses an example shown in [Associate for a single-valued navigation property](#associate-for-a-single-valued-navigation-property).

```ts
let new_testid = "00000000-0000-0000-0000-000000000001";

let request: DynamicsWebApi.AssociateSingleValuedRequest = {
    collection: "new_tests",
    primaryKey: new_testid,
    navigationProperty: "new_ParentLead"
}

await dynamicsWebApi.disassociateSingleValued(request);
//does not have any return value
```

JavaScript sample is the same only without a request type declaration.

### Fetch XML Request

#### TypeScript

```ts
//declaring interface for an Account entity (declaration can be done in d.ts file)
interface Account {
    accountid?: string,
    name?: string
}

//build a fetch xml
let fetchXml = '<fetch mapping="logical">' +
                    '<entity name="account">' +
                        '<attribute name="accountid"/>' +
                        '<attribute name="name"/>' +
                    '</entity>' +
               '</fetch>';

let request: DynamicsWebApi.FetchXmlRequest = {
    collection: "accounts",
    fetchXml: fetchXml
}

let result = await dynamicsWebApi.fetch<Account>(request);
//do something with results here; access records result.value[0].accountid 
```

#### JavaScript

```js
//build a fetch xml
let fetchXml = '<fetch mapping="logical">' +
                    '<entity name="account">' +
                        '<attribute name="accountid"/>' +
                        '<attribute name="name"/>' +
                    '</entity>' +
               '</fetch>';

let request = {
    collection: "accounts",
    fetchXml: fetchXml
}

let result = await dynamicsWebApi.fetch(request);
//do something with results here; access records result.value[0].accountid 
```

#### Paging

```ts
//build a fetch xml
let fetchXml = '<fetch mapping="logical">' +
                    '<entity name="account">' +
                        '<attribute name="accountid"/>' +
                        '<attribute name="name"/>' +
                    '</entity>' +
               '</fetch>';

let request: DynamicsWebApi.FetchXmlRequest = {
    collection: "accounts",
    fetchXml: fetchXml
}

let page1 = await dynamicsWebApi.fetch<Account>(request);
//do something with results here; access records page1.value[0].accountid

request.pageNumber = page1.PagingInfo.nextPage;
request.pagingCookie = page1.PagingInfo.cookie;

let page2 = await dynamicsWebApi.fetch<Account>(request);
//do something with results here; access records page2.value[0].accountid

request.pageNumber = page2.PagingInfo.nextPage;
request.pagingCookie = page2.PagingInfo.cookie;

let page3 = await dynamicsWebApi.fetch<Account>(request);
//and so on... or use a recoursive loop.
```

#### Fetch All records

The following function executes a FetchXml request and goes through all pages automatically:

```ts
let fetchXml = '<fetch mapping="logical">' +
                    '<entity name="account">' +
                        '<attribute name="accountid"/>' +
                        '<attribute name="name"/>' +
                    '</entity>' +
               '</fetch>';

let result = await dynamicsWebApi.fetchAll<Account>({
    collection: "accounts", 
    fetchXml: fetchXml
});
//do something with results here; access records result.value[0].accountid
```

### Execute Web API functions

#### Bound functions

**TypeScript**

```ts
//declaring needed types for the Function (types can be declared in *.d.ts file), if needed
enum UserResponse {
    Basic = 0,
    Local = 1,
    Deep = 2,
    Global = 3
}

interface RolePrivilege {
    Depth: UserResponse,
    PrivilegeId: string,
    BusinessUnitId: string
}

interface RetrieveTeamPrivilegesResponse {
    RolePrivileges: RolePrivilege[]
}

let teamId = "00000000-0000-0000-0000-000000000001";

let request: DynamicsWebApi.BoundFunctionRequest = {
    id: teamId,
    collection: "teams",
    functionName: "Microsoft.Dynamics.CRM.RetrieveTeamPrivileges"
}

let result = await dynamicsWebApi.executeBoundFunction<RetrieveTeamPrivilegesResponse>(request);
//do something with a result
```

**JavaScript**

```js
let teamId = "00000000-0000-0000-0000-000000000001";

let request = {
    id: teamId,
    collection: "teams",
    functionName: "Microsoft.Dynamics.CRM.RetrieveTeamPrivileges"
}

let result = await dynamicsWebApi.executeBoundFunction(request);
//do something with a result
```

#### Unbound functions

**TypeScript**

```ts
//declaring needed types for the Function (types can be declared in *.d.ts file), if needed
interface GetTimeZoneCodeByLocalizedNameResponse {
    TimeZoneCode: number
}

let parameters = {
    LocalizedStandardName: "Pacific Standard Time",
    LocaleId: 1033
};

let request: DynamicsWebApi.UnboundFunctionRequest = {
    parameters: parameters,
    functionName: "GetTimeZoneCodeByLocalizedName"
}

let result = await dynamicsWebApi.executeUnboundFunction<GetTimeZoneCodeByLocalizedNameResponse>(request);
let timeZoneCode = result.TimeZoneCode;
```

**JavaScript**

```js
let parameters = {
    LocalizedStandardName: "Pacific Standard Time",
    LocaleId: 1033
};

let request = {
    parameters: parameters,
    functionName: "GetTimeZoneCodeByLocalizedName"
}

let result = await dynamicsWebApi.executeUnboundFunction(request);
let timeZoneCode = result.TimeZoneCode;
```

### Execute Web API actions

#### Bound actions

```js
var queueId = "00000000-0000-0000-0000-000000000001";
var letterActivityId = "00000000-0000-0000-0000-000000000002";
var actionRequest = {
    Target: {
        activityid: letterActivityId,
        "@odata.type": "Microsoft.Dynamics.CRM.letter"
    }
};
dynamicsWebApi.executeBoundAction(queueId, "queues", "Microsoft.Dynamics.CRM.AddToQueue", actionRequest)
    .then(function (result) {
        var queueItemId = result.QueueItemId;
    })
    .catch(function (error) {
        //catch an error
    });
```

#### Unbound actions

```js
var opportunityId = "b3828ac8-917a-e511-80d2-00155d2a68d2";
var actionRequest = {
    Status: 3,
    OpportunityClose: {
        subject: "Won Opportunity",

        //DynamicsWebApi will add full url if the property contains @odata.bind suffix
        //but it is also possible to specify a full url to the entity record
        "opportunityid@odata.bind": "opportunities(" + opportunityId + ")"
    }
};
dynamicsWebApi.executeUnboundAction("WinOpportunity", actionRequest).then(function () {
    //success
}).catch(function (error) {
    //catch an error
});
```

## Execute Batch Operations

`version 1.5.0+`

Batch requests bundle multiple operations into a single one and have the following advantages:

* Reduces a number of requests sent to the Web API server. `Each user is allowed up to 60,000 API requests, per organization instance, within five minute sliding interval.` [More Info](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/api-limits)
* Provides a way to run multiple operations in a single transaction. If any operation that changes data (within a single changeset) fails all completed ones will be rolled back.
* All operations within a batch request run consequently (FIFO).

DynamicsWebApi provides a straightforward way to execute Batch operations which may not always simple to compose. 
The following example bundles 2 retrieve multiple operations and an update:

```js

//when you want to start a batch operation call the following function:
//it is important to call it, otherwise all operations below will be executed right away.
dynamicsWebApi.startBatch();

//call necessary operations just like you would normally do.
//these calls will be converted into a single batch request
dynamicsWebApi.retrieveMultiple('accounts');
dynamicsWebApi.update('00000000-0000-0000-0000-000000000002', 'contacts', { firstname: "Test", lastname: "Batch!" });
dynamicsWebApi.retrieveMultiple('contacts');

//execute a batch request:
dynamicsWebApi.executeBatch()
    .then(function (responses) {
        //'responses' is an array of responses of each individual request
        //they have the same sequence as the calls between startBatch() and executeBatch()
        //in this case responses.length is 3

        //dynamicsWebApi.retrieveMultiple response:
        var accounts = responses[0];
        //dynamicsWebApi.update response
        var isUpdated = responses[1]; //should be 'true'
        //dynamicsWebApi.retrieveMultiple response:
        var contacts = responses[2]; //will contain an updated contact

    }).catch(function (error) {
        //catch error here
    });

```

The next example shows how to run multiple operations in a single transaction which means if at least one operation fails all completed ones will be rolled back which ensures a data consistency.

```js

//for example, a user did a checkout and we need to create two orders

var order1 = {
    name: '1 year membership',
    'customerid_contact@odata.bind': 'contacts(00000000-0000-0000-0000-000000000001)'
};

var order2 = {
    name: 'book',
    'customerid_contact@odata.bind': 'contacts(00000000-0000-0000-0000-000000000001)'
};

dynamicsWebApi.startBatch();

dynamicsWebApi.create(order1, 'salesorders');
dynamicsWebApi.create(order2, 'salesorders');

dynamicsWebApi.executeBatch().then(function (responses) {
    var salesorderId1 = responses[0];
    var salesorderId2 = responses[1];
}).catch(function (error) {
    //catch error here
    //all completed operations will be rolled back
    alert('Cannot complete a checkout. Please try again later.');
});

```

**Important!** Developers who use DynamicsWebApi with callbacks do not need to pass `successCallback` and `errorCallback` in an individual operation when `startBatch()` is called, 
just pass `null` if you need to add additional parameters in the request, 
for example: `dynamicsWebApi.deleteRecord('00000000-0000-0000-0000-000000000001', 'contacts', null, null, 'firstname')`.

### Use Content-ID to reference requests in a Change Set

`version 1.5.6+`

You can reference a request in a Change Set. For example, if you want to create related entities in a single batch request:

```js
var order = {
    name: '1 year membership'
};

var contact = {
    firstname: 'John',
    lastname: 'Doe'
};

dynamicsWebApi.startBatch();
dynamicsWebApi.createRequest({ entity: order, collection: 'salesorders', contentId: '1' });
dynamicsWebApi.createRequest({ entity: contact, collection: 'customerid_contact', contentId: '$1' });

dynamicsWebApi.executeBatch()
    .then(function (responses) {
        var salesorderId = responses[0];
        //responses[1]; is undefined <- CRM Web API limitation
    }).catch(function (error) {
        //catch error here
    });

```

Note that the second response does not have a returned value, it is a CRM Web API limitation.

**Important!** DynamicsWebApi automatically assigns value to a `Content-ID` if it is not provided, therefore, please set your `Content-ID` value less than 100000.

### Use Content-ID inside a request payload

`version 1.5.7+`

Another option to make the same request is to use `Content-ID` reference inside a request payload as following:

```js

var contact = {
    firstname: 'John',
    lastname: 'Doe'
};

var order = {
    name: '1 year membership',
    //reference a request in a navigation property
    'customerid_contact@odata.bind': '$1'
};

dynamicsWebApi.startBatch();
dynamicsWebApi.createRequest({ entity: contact, collection: 'contacts', contentId: '1' });
dynamicsWebApi.createRequest({ entity: order, collection: 'salesorders' });

dynamicsWebApi.executeBatch()
    .then(function (responses) {
        //in this case both ids exist in a response
        //which makes it a preferred method
        var contactId = responses[0];
        var salesorderId = responses[1];
    }).catch(function (error) {
        //catch error here
    });

```

#### Limitations

Currently, there are some limitations in DynamicsWebApi Batch Operations:

* Operations that use pagination to recursively retrieve all records cannot be used in a 'batch mode'. These include: `retrieveAll`, `retrieveAllRequest`, `countAll`, `fetchAll`, `executeFetchXmlAll`.
You will get an error saying that the operation is incompatible with a 'batch mode'.
* **Does not apply to `v.1.6.5+`**: The following limitation is for external applications (working outside D365 CE forms). `useEntityNames` may not work in a 'batch mode' if it is set to `true`. 
To make sure that it works, please execute any operation before calling `dynamicsWebApi.startBatch()` so that it caches all entity names, for example: `dynamicsWebApi.count('account')`.

There are also out of the box Web API limitations for batch operations:

* Batch requests can contain up to 1000 individual requests and cannot contain other batch requests.
* The `odata.continue-on-error` preference is not supported by the Web API. Any error that occurs in the batch will stop the processing of the remainder of the batch.

You can find an official documentation that covers Web API batch requests here: [Execute batch operations using the Web API](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/webapi/execute-batch-operations-using-web-api).

## Working with Metadata Definitions

`Version 1.4.3+`

Before working with metadata read [the following section from Microsoft Documentation](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/webapi/use-web-api-metadata).

### Create Entity

```js
var entityDefinition = {
    "@odata.type": "Microsoft.Dynamics.CRM.EntityMetadata",
    "Attributes": [
    {
        "AttributeType": "String",
        "AttributeTypeName": {
            "Value": "StringType"
        },
        "Description": {
            "@odata.type": "Microsoft.Dynamics.CRM.Label",
            "LocalizedLabels": [{
                "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                "Label": "Type the name of the bank account",
                "LanguageCode": 1033
            }]
        },
        "DisplayName": {
            "@odata.type": "Microsoft.Dynamics.CRM.Label",
            "LocalizedLabels": [{
                "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                "Label": "Account Name",
                "LanguageCode": 1033
            }]
        },
        "IsPrimaryName": true,
        "RequiredLevel": {
            "Value": "None",
            "CanBeChanged": true,
            "ManagedPropertyLogicalName": "canmodifyrequirementlevelsettings"
        },
        "SchemaName": "new_AccountName",
        "@odata.type": "Microsoft.Dynamics.CRM.StringAttributeMetadata",
        "FormatName": {
            "Value": "Text"
        },
        "MaxLength": 100
    }],
    "Description": {
        "@odata.type": "Microsoft.Dynamics.CRM.Label",
        "LocalizedLabels": [{
            "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
            "Label": "An entity to store information about customer bank accounts",
            "LanguageCode": 1033
        }]
    },
    "DisplayCollectionName": {
        "@odata.type": "Microsoft.Dynamics.CRM.Label",
        "LocalizedLabels": [{
            "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
            "Label": "Bank Accounts",
            "LanguageCode": 1033
        }]
    },
    "DisplayName": {
        "@odata.type": "Microsoft.Dynamics.CRM.Label",
        "LocalizedLabels": [{
            "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
            "Label": "Bank Account",
            "LanguageCode": 1033
        }]
    },
    "HasActivities": false,
    "HasNotes": false,
    "IsActivity": false,
    "OwnershipType": "UserOwned",
    "SchemaName": "new_BankAccount"
};

dynamicsWebApi.createEntity(entityDefinition).then(function(entityId){
    //entityId is newly created entity id (MetadataId)
}).catch(function(error){
    //catch an error
})
```

### Retrieve Entity

Entity Metadata can be retrieved by either Primary Key (**MetadataId**) or by an Alternate Key (**LogicalName**). [More Info](https://msdn.microsoft.com/en-us/library/mt788314.aspx#bkmk_byName)

```js
var entityKey = '00000000-0000-0000-0000-000000000001';
//or you can use an alternate key:
//var entityKey = "LogicalName='new_accountname'";
dynamicsWebApi.retrieveEntity(entityKey, ['SchemaName', 'LogicalName']).then(function(entityMetadata){
    var schemaName = entityMetadata.SchemaName;
}).catch(function(error){
    //catch an error
});
```

### Update Entity

Microsoft recommends to make changes in the entity metadata that has been priorly retrieved to avoid any mistake. I would also recommend to read information about **MSCRM.MergeLabels** header prior updating metadata. More information about the header can be found [here](https://msdn.microsoft.com/en-us/library/mt593078.aspx#Anchor_2).

**Important!** Make sure you set **`MetadataId`** property when you update the metadata, DynamicsWebApi use it as a primary key for the EntityDefinition record.

```js
var entityKey = "LogicalName='new_accountname'";
dynamicsWebApi.retrieveEntity(entityKey).then(function(entityMetadata){
    //1. change label
    entityMetadata.DispalyName.LocalizedLabels[0].Label = 'New Bank Account';
    //2. update metadata
    return dynamicsWebApi.updateEntity(entityMetadata);
}).catch(function(error){
    //catch an error
});
```

**Important!** When you update an entity, you must publish changes in CRM. [More Info](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/customize-dev/publish-customizations)

### Retrieve Multiple Entities

```js
dynamicsWebApi.retrieveEntities(['LogicalName'], "OwnershipType eq Microsoft.Dynamics.CRM.OwnershipTypes'UserOwned'").then(function(response){
    var firstLogicalName = response.value[0].LogicalName;
}).catch(function(error){
    //catch an error
});
```

### Create Attribute

```js
var entityKey = '00000000-0000-0000-0000-000000000001';
var attributeDefinition = {
    "AttributeType": "Money",
    "AttributeTypeName": {
        "Value": "MoneyType"
    },
    "Description": {
        "@odata.type": "Microsoft.Dynamics.CRM.Label",
        "LocalizedLabels": [{
            "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
            "Label": "Enter the balance amount",
            "LanguageCode": 1033
        }]
    },
    "DisplayName": {
        "@odata.type": "Microsoft.Dynamics.CRM.Label",
        "LocalizedLabels": [{
            "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
            "Label": "Balance",
            "LanguageCode": 1033
        }]
    },
    "RequiredLevel": {
        "Value": "None",
        "CanBeChanged": true,
        "ManagedPropertyLogicalName": "canmodifyrequirementlevelsettings"
    },
    "SchemaName": "new_Balance",
    "@odata.type": "Microsoft.Dynamics.CRM.MoneyAttributeMetadata",
    "PrecisionSource": 2
};

dynamicsWebApi.createAttribute(entityKey, attributeDefinition).then(function(attributeId){
    //attributeId is a PrimaryKey (MetadataId) for newly created attribute
}).catch(function(error){
    //catch an error
});
```

### Retrieve Attribute

Attribute Metadata can be retrieved by either Primary Key (**MetadataId**) or by an Alternate Key (**LogicalName**). [More Info](https://msdn.microsoft.com/en-us/library/mt788314.aspx#bkmk_byName)

The following example will retrieve only common properties available in [AttributeMetadata](https://msdn.microsoft.com/en-us/library/mt607551.aspx) entity.

```js
var entityKey = '00000000-0000-0000-0000-000000000001';
//or you can use an alternate key:
//var entityKey = "LogicalName='new_accountname'";
var attributeKey = '00000000-0000-0000-0000-000000000002';
//or you can use an alternate key:
//var attributeKey = "LogicalName='new_balance'";
dynamicsWebApi.retrieveAttribute(entityKey, attributeKey, ['SchemaName']).then(function(attributeMetadata){
    var schemaName = attributeMetadata.SchemaName;
}).catch(function(error){
    //catch an error
});
```

Use parameter in the function to cast the attribute to a specific type.

```js
var entityKey = '00000000-0000-0000-0000-000000000001';
var attributeKey = '00000000-0000-0000-0000-000000000002';
dynamicsWebApi.retrieveAttribute(entityKey, attributeKey, ['SchemaName'], 'Microsoft.Dynamics.CRM.MoneyAttributeMetadata')
    .then(function(attributeMetadata){
        var schemaName = attributeMetadata.SchemaName;
    }).catch(function(error){
        //catch an error
    });
```

### Update Attribute

**Important!** Make sure you set **`MetadataId`** property when you update the metadata, DynamicsWebApi use it as a primary key for the EntityDefinition record.

The following example will update only common properties availible in [AttributeMetadata](https://msdn.microsoft.com/en-us/library/mt607551.aspx) entity. If you need to update specific properties of Attributes with type that inherit from the AttributeMetadata you will need to cast the attribute to the specific type. [More Info](https://msdn.microsoft.com/en-us/library/mt607522.aspx#Anchor_4)

```js
var entityKey = "LogicalName='new_accountname'";
var attributeKey = "LogicalName='new_balance'";
dynamicsWebApi.retrieveAttribute(entityKey, attributeKey).then(function(attributeMetadata){
    //1. change label
    attributeMetadata.DispalyName.LocalizedLabels[0].Label = 'New Balance';
    //2. update metadata
    return dynamicsWebApi.updateAttribute(entityKey, attributeMetadata);
}).catch(function(error){
    //catch an error
});
```

To cast a property to a specific type use a parameter in the function.

```js
var entityKey = "LogicalName='new_accountname'";
var attributeKey = "LogicalName='new_balance'";
var attributeType = 'Microsoft.Dynamics.CRM.MoneyAttributeMetadata';
dynamicsWebApi.retrieveAttribute(entityKey, attributeKey, attributeType).then(function(attributeMetadata){
    //1. change label
    attributeMetadata.DispalyName.LocalizedLabels[0].Label = 'New Balance';
    //2. update metadata
    return dynamicsWebApi.updateAttribute(entityKey, attributeMetadata, attributeType);
}).catch(function(error){
    //catch an error
});
```

**Important!** Make sure you include the attribute type in the update function as well.

**Important!** When you update an attribute, you must publish changes in CRM. [More Info](https://docs.microsoft.com/en-us/dynamics365/customer-engagement/developer/customize-dev/publish-customizations)

### Retrieve Multiple Attributes

The following example will retrieve only common properties available in [AttributeMetadata](https://msdn.microsoft.com/en-us/library/mt607551.aspx) entity.

```js
var entityKey = "LogicalName='new_accountname'";
dynamicsWebApi.retrieveAttributes(entityKey).then(function(response){
    var firstAttribute = response.value[0];
}).catch(function(error){
    //catch an error
});
```

To retrieve only attributes of a specific type use a parameter in a function:

```js
var entityKey = "LogicalName='new_accountname'";
dynamicsWebApi.retrieveAttributes(entityKey, 'Microsoft.Dynamics.CRM.MoneyAttributeMetadata').then(function(response){
    var firstAttribute = response.value[0];
}).catch(function(error){
    //catch an error
});
```

### Use requests to query Entity and Attribute metadata

You can also use common request functions to create, retrieve and update entity and attribute metadata. Just use the following rules:

1. Always set `collection: 'EntityDefinitions'`.
2. To retrieve a specific **entity metadata** by a Primary or Alternate Key use `key` property. For example: `key: 'LogicalName="account"'`.
3. To get attributes, set `navigationProperty: 'Attributes'`.
4. To retrieve a specific **attribute metadata** by Primary or Alternate Key use `navigationPropertyKey`. For example: `navigationPropertyKey: '00000000-0000-0000-0000-000000000002'`.
5. During entity or attribute metadata update you can use `mergeLabels` property to set **MSCRM.MergeLabels** attribute. By default `mergeLabels: false`.
6. To send entity or attribute definition use `entity` property.

#### Examples

Retrieve entity metadata with attributes (with common properties):

```js
var request = {
    collection: 'EntityDefinitions',
    key: '00000000-0000-0000-0000-000000000001',
    select: ['LogicalName', 'SchemaName'],
    expand: 'Attributes'
};

dynamicsWebApi.retrieveRequest(request).then(function(entityMetadata){
    var attributes = entityMetadata.Attributes;
}).catch(function(error){
    //catch an error
});
```

Retrieve attribute metadata and cast it to the StringType:

```js
var request = {
    collection: 'EntityDefinitions',
    key: 'LogicalName="account"',
    navigationProperty: 'Attributes',
    navigationPropertyKey: 'LogicalName="firstname"',
    metadataAttributeType: 'Microsoft.Dynamics.CRM.StringAttributeMetadata'
};

dynamicsWebApi.retrieveRequest(request).then(function(attributeMetadata){
    var displayNameDefaultLabel = attributeMetadata.DisplayName.LocalizedLabels[0].Label;
}).catch(function(error){
    //catch an error
});
```

Update entity metadata with **MSCRM.MergeLabels** header set to `true`:

```js
var request = {
    collection: 'EntityDefinitions',
    key: 'LogicalName="account"'
};

dynamicsWebApi.retrieveRequest(request).then(function(entityMetadata){
    //1. change label
    entityMetadata.DisplayName.LocalizedLabels[0].Label = 'Organization';
    //2. configure update request
    var updateRequest = {
        collection: 'EntityDefinitions',
        key: entityMetadata.MetadataId,
        mergeLabels: true,
        entity: entityMetadata
    };
    //3. call update request
    return dynamicsWebApi.updateRequest(updateRequest);
}).catch(function(error){
    //catch an error
});

//it is the same as:
dynamicsWebApi.retrieveEntity('LogicalName="account"').then(function(entityMetadata){
    //1. change label
    entityMetadata.DisplayName.LocalizedLabels[0].Label = 'Organization';
    //2. call update request
    return dynamicsWebApi.updateEntity(entityMetadata, true);
}).catch(function(error){
    //catch an error
});
```

### Create Relationship

```js
var newRelationship = {
    "SchemaName": "dwa_contact_dwa_dynamicswebapitest",
    "@odata.type": "Microsoft.Dynamics.CRM.OneToManyRelationshipMetadata",
    "AssociatedMenuConfiguration": {
        "Behavior": "UseCollectionName",
        "Group": "Details",
        "Label": {
            "@odata.type": "Microsoft.Dynamics.CRM.Label",
            "LocalizedLabels": [
             {
                 "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                 "Label": "DWA Test",
                 "LanguageCode": 1033
             }
            ],
            "UserLocalizedLabel": {
                "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                "Label": "DWA Test",
                "LanguageCode": 1033
            }
        },
        "Order": 10000
    },
    "CascadeConfiguration": {
        "Assign": "Cascade",
        "Delete": "Cascade",
        "Merge": "Cascade",
        "Reparent": "Cascade",
        "Share": "Cascade",
        "Unshare": "Cascade"
    },
    "ReferencedAttribute": "contactid",
    "ReferencedEntity": "contact",
    "ReferencingEntity": "dwa_dynamicswebapitest",
    "Lookup": {
        "AttributeType": "Lookup",
        "AttributeTypeName": {
            "Value": "LookupType"
        },
        "Description": {
            "@odata.type": "Microsoft.Dynamics.CRM.Label",
            "LocalizedLabels": [
             {
                 "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                 "Label": "The owner of the test",
                 "LanguageCode": 1033
             }
            ],
            "UserLocalizedLabel": {
                "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                "Label": "The owner of the test",
                "LanguageCode": 1033
            }
        },
        "DisplayName": {
            "@odata.type": "Microsoft.Dynamics.CRM.Label",
            "LocalizedLabels": [
             {
                 "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                 "Label": "DWA Test Owner",
                 "LanguageCode": 1033
             }
            ],
            "UserLocalizedLabel": {
                "@odata.type": "Microsoft.Dynamics.CRM.LocalizedLabel",
                "Label": "DWA Test Owner",
                "LanguageCode": 1033
            }
        },
        "RequiredLevel": {
            "Value": "ApplicationRequired",
            "CanBeChanged": true,
            "ManagedPropertyLogicalName": "canmodifyrequirementlevelsettings"
        },
        "SchemaName": "dwa_TestOwner",
        "@odata.type": "Microsoft.Dynamics.CRM.LookupAttributeMetadata"
    }
};

dynamicsWebApi.createRelationship(newRelationship).then(function (relationshipId) {
    //relationshipId is a PrimaryKey (MetadataId) for a newly created relationship
}).catch(function (error) {
    //catch errors
});
```
### Update Relationship

**Important!** Make sure you set **`MetadataId`** property when you update the metadata, DynamicsWebApi use it as a primary key for the EntityDefinition record.

```js
var metadataId = '10cb680e-b6a7-e811-816a-480fcfe97e21';

dynamicsWebApi.retrieveRelationship(metadataId).then(function (relationship) {
    relationship.AssociatedMenuConfiguration.Label.LocalizedLabels[0].Label = "New Label";
    return dynamicsWebApi.updateRelationship(relationship);
}).then(function (updateResponse) {
    //check update response
}).catch(function (error) {
    //catch errors
});
```

### Delete Relationship

```js
var metadataId = '10cb680e-b6a7-e811-816a-480fcfe97e21';

dynamicsWebApi.deleteRelationship(metadataId).then(function (isDeleted) {
    //isDeleted should be true
}).catch(function (error) {
    //catch errors
});
```

### Retrieve Relationship

```js
var metadataId = '10cb680e-b6a7-e811-816a-480fcfe97e21';

dynamicsWebApi.retrieveRelationship(metadataId).then(function (relationship) {
    //work with a retrieved relationship
}).catch(function (error) {
    //catch errors
});
```

You can also cast a relationship into a specific type:

```js
var metadataId = '10cb680e-b6a7-e811-816a-480fcfe97e21';
var relationshipType = 'Microsoft.Dynamics.CRM.OneToManyRelationshipMetadata';
dynamicsWebApi.retrieveRelationship(metadataId, relationshipType).then(function (relationship) {
    //work with a retrieved relationship
}).catch(function (error) {
    //catch errors
});
```

### Retrieve Multiple Relationships

```js
dynamicsWebApi.retrieveRelationships(['SchemaName', 'MetadataId'], "ReferencedEntity eq 'account'")
.then(function (relationship) {
    //work with a retrieved relationship
}).catch(function (error) {
    //catch errors
});
```

### Create Global Option Set

`version 1.4.6+`

```js
var optionSetDefinition = {
    "@odata.type": "Microsoft.Dynamics.CRM.OptionSetMetadata",
    IsCustomOptionSet: true,
    IsGlobal: true,
    IsManaged: false,
    Name: "new_customglobaloptionset",
    OptionSetType: "Picklist",
    Options: [{
        Value: 0,
        Label: {
            LocalizedLabels: [{
                Label: "Label 1", LanguageCode: 1033
            }],
            UserLocalizedLabel: {
                Label: "Label 1", LanguageCode: 1033
            }
        },
        Description: {
            LocalizedLabels: [],
            UserLocalizedLabel: null
        }
    }, {
        Value: 1,
        Label: {
            LocalizedLabels: [{
                Label: "Label 2", LanguageCode: 1033
            }],
            UserLocalizedLabel: {
                Label: "Label 2", LanguageCode: 1033
            }
        },
        Description: {
            LocalizedLabels: [],
            UserLocalizedLabel: null
        }
    }],
    Description: {
        LocalizedLabels: [{
            Label: "Description to the Global Option Set.", LanguageCode: 1033
        }],
        UserLocalizedLabel: {
            Label: "Description to the Global Option Set.", LanguageCode: 1033
        }
    },
    DisplayName: {
        LocalizedLabels: [{
            Label: "Display name to the Custom Global Option Set.", LanguageCode: 1033
        }],
        UserLocalizedLabel: {
            Label: "Display name to the Custom Global Option Set.", LanguageCode: 1033
        }
    },
    IsCustomizable: {
        Value: true, "CanBeChanged": true, ManagedPropertyLogicalName: "iscustomizable"
    }
};

dynamicsWebApi.createGlobalOptionSet(optionSetDefinition).then(function (id) {
    //metadata id
}).catch(function (error) {
    //catch error here
});
```

### Update Global Option Set

`version 1.4.6+`

**Important!** Publish your changes after update, otherwise a label won't be modified.

```js
var key = '6e133d25-abd1-e811-816e-480fcfeab9c1';
//or
key = "Name='new_customglobaloptionset'";

dynamicsWebApi.retrieveGlobalOptionSet(key).then(function (response) {
    response.DisplayName.LocalizedLabels[0].Label = "Updated Display name to the Custom Global Option Set.";
    return dynamicsWebApi.updateGlobalOptionSet(response);
}).then(function (response) {
    //check if it was updated
}).catch (function (error) {
    //catch error here
});
```

### Delete Global Option Set

`version 1.4.6+`

```js
var key = '6e133d25-abd1-e811-816e-480fcfeab9c1';
//or
key = "Name='new_customglobaloptionset'";

dynamicsWebApi.deleteGlobalOptionSet(key).then(function (response) {
    //check if it was deleted
}).catch(function (error) {
    //catch error here
});
```

### Retrieve Global Option Set

`version 1.4.6+`

```js
var key = '6e133d25-abd1-e811-816e-480fcfeab9c1';
//or
key = "Name='new_customglobaloptionset'";

dynamicsWebApi.retrieveGlobalOptionSet(key).then(function (response) {
    //response.DisplayName.LocalizedLabels[0].Label
}).catch (function (error) {
    //catch error here
});

//select specific attributes
//select specific attributes
dynamicsWebApi.retrieveGlobalOptionSet(key, null, ['Name']).then(function (response) {
    //response.DisplayName.LocalizedLabels[0].Label
}).catch (function (error) {
    //catch error here
});

//Options attribute exists only in OptionSetMetadata, therefore we need to cast to it
dynamicsWebApi.retrieveGlobalOptionSet(key, 'Microsoft.Dynamics.CRM.OptionSetMetadata', ['Name', 'Options']).then(function (response) {
    //response.DisplayName.LocalizedLabels[0].Label
}).catch (function (error) {
    //catch error here
});
```

### Retrieve Multiple Global Option Sets

`version 1.4.6+`

```js
dynamicsWebApi.retrieveGlobalOptionSets().then(function (response) {
	var optionSet = response.value[0]; //first global option set
}).catch (function (error) {
    //catch error here
});

//select specific attributes
dynamicsWebApi.retrieveGlobalOptionSets('Microsoft.Dynamics.CRM.OptionSetMetadata', ['Name', 'Options']).then(function (response) {
	var optionSet = response.value[0]; //first global option set
}).catch (function (error) {
    //catch error here
});
```

## Formatted Values and Lookup Properties

Starting from version 1.3.0 it became easier to access formatted values for properties and lookup data in response objects. 
DynamicsWebApi automatically creates aliases for each property that contains a formatted value or lookup data.
For example:

```js
//before v.1.3.0 a formatted value for account.donotpostalmail field could be accessed as following:
var doNotPostEmailFormatted = response['donotpostalmail@OData.Community.Display.V1.FormattedValue'];

//starting with v.1.3.0 it can be simplified
doNotPostEmailFormatted = response.donotpostalmail_Formatted;

//same for lookup data
//before v.1.3.0
var customerName = response['_customerid_value@OData.Community.Display.V1.FormattedValue'];
var customerEntityLogicalName = response['_customerid_value@Microsoft.Dynamics.CRM.lookuplogicalname'];
var customerNavigationProperty = response['_customerid_value@Microsoft.Dynamics.CRM.associatednavigationproperty'];

//starting with v.1.3.0
customerName = response._customerid_value_Formatted;
customerEntityLogicalName = response._customerid_value_LogicalName;
customerNavigationProperty = response._customerid_value_NavigationProperty;
```

If you still want to use old properties you can do so, they are not removed from the response, so it does not break your existing functionality.

As you have already noticed formatted and lookup data values are accesed by adding a particular suffix to a property name, 
the following table summarizes it.

OData Annotation | Property Suffix
------------ | -------------
`@OData.Community.Display.V1.FormattedValue` | `_Formatted`
`@Microsoft.Dynamics.CRM.lookuplogicalname` | `_LogicalName`
`@Microsoft.Dynamics.CRM.associatednavigationproperty` | `_NavigationProperty`

## Using Alternate Keys
Starting from version 1.3.4, you can use alternate keys to Update, Upsert, Retrieve and Delete records. [More Info](https://msdn.microsoft.com/en-us/library/mt607871.aspx#Retrieve%20using%20an%20alternate%20key)

### Basic usage

```js
var alternateKey = "key='keyValue'"; 
//or var alternateKey = "key='keyValue',anotherKey='keyValue2'";

//perform a retrieve operaion
dynamicsWebApi.retrieve(alternateKey, "leads", ["fullname", "subject"]).then(function (record) {
    //do something with a record here
})
.catch(function (error) {
    //catch an error
});
```

### Advanced using Request Object

Please use `key` instead of `id` for all requests that you make using DynamicsWebApi starting from `v.1.3.4`.

Please note, that `id` field is not removed from the library, so all your existing scripts will work without any issue.

```js
var request = {
    key: "alternateKey='keyValue'",
    collection: 'leads',
    select: ['fullname', 'subject']
};

dynamicsWebApi.retrieveRequest(request).then(function (record) {
    //do something with a record
})
.catch(function (error) {
    //if the record has not been found the error will be thrown
});
```

`key` can be used as a primary key (id):

```js
var request = {
    key: '00000000-0000-0000-0000-000000000001',
    collection: 'leads',
    select: ['fullname', 'subject']
};

dynamicsWebApi.retrieveRequest(request).then(function (record) {
    //do something with a record
})
.catch(function (error) {
    //if the record has not been found the error will be thrown
});
```

## Making requests using Entity Logical Names

Starting from version 1.4.0, it is possible to make requests using Entity Logical Names (for example: `account`, instead of `accounts`).
There's a small perfomance impact when this feature is used **outside CRM/D365 Web Resources**: DynamicsWebApi makes a request to
Entity Metadata and retrieves LogicalCollectionName and LogicalName for all entities during **the first call to Web Api** on the page.

To enable this feature set `useEntityNames: true` in DynamicsWebApi config.

```js
var dynamicsWebApi = new DynamicsWebApi({ useEntityNames: true });

//make request using entity names
dynamicsWebApi.retrieve(leadId, 'lead', ['fullname', 'subject']).then(function (record) {
    //do something with a record here
})
.catch(function (error) {
    //catch an error
});

//this will also work in request functions, even though the name of the property is a collection

var request = {
    collection: 'lead',
    key: leadId,
    select:  ['fullname', 'subject']
};

dynamicsWebApi.retrieveRequest(request).then(function (record) {
    //do something with a record here
})
.catch(function (error) {
    //catch an error
});
```

This feature also applies when you set a navigation property and provide an entity name in the value:

```js
var account = {
    name: 'account name',
   'primarycontactid@odata.bind': 'contact(00000000-0000-0000-0000-000000000001)'
}

dynamicsWebApi.create(account, 'account').then(function(accountId)){
    //newly created accountId
}).catch(function (error) {
    //catch error here
});
```

In the example above, entity names will be replaced with collection names: `contact` with `contacts`, `account` with `accounts`.
This happens, because DynamicsWebApi automatically checks all properties that end with `@odata.bind` or `@odata.id`. 
Thus, there may be a case when those properties are not used but you still need a collection name instead of an entity name.
Please use the following method to get a collection name from a cached entity metadata:

```js
//IMPORTANT! collectionName will be null if there was no call to Web API prior to that
//this restriction does not apply if DynamicsWebApi used inside CRM/D365
var collectionName = dynamicsWebApi.utility.getCollectionName('account');
```

Please note, everything said above will happen only if you set `useEntityNames: true` in the DynamicsWebApi config.

## Using TypeScript Declaration Files

TypeScript declaration files `d.ts` added with v.1.5.3. 
If you are not familiar with declaration files, these files are used to provide TypeScript type information about an API that's written in JavaScript.
You want to consume those from your TypeScript code. [Quote](https://stackoverflow.com/a/21247316/2042071)

At this moment, DynamicsWebApi declaration files do not exist in a [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) repository but it is in my plans to upload them there.

### Node.Js

If you are using Node.Js with TypeScript, declarations will be fetched with an NPM package during an installation or an update process.
At the top of a necessary `.ts` file add the following:

```ts
import * as DynamicsWebApi from "dynamics-web-api";
//for CommonJS:
//import DynamicsWebApi = require("dynamics-web-api");
```

### Dynamics 365 web resource
If you are developing CRM Web Resources with TypeScript, you will need to download a necessary `d.ts` file manually from the following folder: [types](/types/).
As you may have noticed `types` folder contains two declaration files: `dynamics-web-api.d.ts` (Promises) and `dynamics-web-api-callbacks.d.ts` (Callbacks) - download the one that you need.
**Do not download both files! Otherwise you will have type declaration conflicts.**
In my web resources project I usually put a declaration file under "./types/" folder. For example:

```
[project root]/
-- src/
  -- form_web_resource.ts
-- types/
  -- dynamics-web-api/
    -- dynamics-web-api-callbacks.d.ts
-- tsconfig.json
```

**Important!** Make sure that you include `types` folder in your `tsconfig.json`:
```
"include": [
	"./src/**/*",
	"./types/**/*"
]
```

### In Progress

- [X] Overloaded functions with rich request options for all Web API operations.
- [X] Get all pages requests, such as: countAll, retrieveMultipleAll, fetchXmlAll and etc. `Implemented in v.1.2.5`
- [X] Web API requests that have long URL (more than 2000 characters) should be automatically converted to batch requests. 
Feature is very convenient for big Fetch XMLs. `Implemented in v.1.2.8`
- [X] "Formatted" values in responses. For instance: Web API splits information about lookup fields into separate properties, 
the config option "formatted" will enable developers to retrieve all information about such fields in a single requests and access it through DynamicsWebApi custom response objects.
- [X] Simplified names for "Formatted" properties. `Implemented in v.1.3.0`
- [X] RUD operations using Alternate Keys. `Implemented in v.1.3.4`
- [X] Duplicate Detection for Web API v.9. `Implemented in v.1.3.4`
- [X] Ability to use entity names instead of collection names. `Implemented in v.1.4.0`
- [X] Entity and Attribute Metadata helpers. `Implemented in v.1.4.3`
- [X] Entity Relationships and Global Option Sets helpers. `Implemented in v.1.4.6`
- [X] Batch requests. `Implemented in v.1.5.0`
- [X] TypeScript declaration files `d.ts` `Added in v.1.5.3`.
- [X] Implement `Content-ID` header to reference a request in a Change Set in a batch operation `Added in v.1.5.6`.
- [X] Change Tracking `Added in v.1.5.11`.
- [ ] Refactoring and conversion to TypeScript - coming with `v.2.0`! Stay tuned!

Many more features to come!

Thank you for your patience and for using DynamcisWebApi!

## JavaScript Promises
Please use the following library that implements [ES6 Promises](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise): [DynamicsWebApi with Promises](/scripts/dynamics-web-api.js).

It is highly recommended to use one of the Promise Polyfills (Yaku, ES6 Promise and etc.) if DynamicsWebApi is intended to be used in the browsers.

## JavaScript Callbacks
Please use the following library that implements Callbacks : [DynamicsWebApi with Callbacks](/scripts/dynamics-web-api-callbacks.js).

## Contributions

First of all, I would like to thank you for using `DynamicsWebApi` library in your Dynamics 365 CE / Common Data Service project, the fact that my project helps someone to achieve their development goals already makes me happy. 

And if you would like to contribute to the project you may do it in multiple ways:
1. Submit an issue/bug if you have encountered one.
2. If you know the root of the issue please feel free to submit a pull request, just make sure that all tests pass and if the fix needs new unit tests, please add one.
3. Let me and community know if you have any ideas or suggestions on how to improve the project by submitting an issue on GitHub, I will label it as a 'future enhancement'.
4. Feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/alexrogov/) and if you wish to let me know how you use `DynamicsWebApi` and what project you are working on, I will be happy to hear about it.
5. If you feel that this project saved your time and you would like to support it, then please feel free to donate: [![PayPal.Me](/extra/paypal.png)](https://paypal.me/alexrogov)

Any contribution is greatly appreciated!