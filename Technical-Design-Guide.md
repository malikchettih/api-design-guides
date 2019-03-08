# Technical Design Guide
Warning: This document is copy/past notes from https://github.corp.dh.com/pages/ARPnP/api-design/ documents.

## Summary

- [1. Rest Principals](#1)
- [2. Naming conventions](#2)
- [3. HTTP Methods](#3)
- [4. HTTP Headers](#4)
- [5. Error Codes & RFC 7807](#5)
- [6. Versioning](#6)
- [7. Navigation, Sorting, paging, filtering and seraching and projection](#7)
- [8. Asynchronous and long running operation](#8)
- [9. Caching](#9)
- [10. Concurrency](#10)
- [11. Tokenization](#11)


## <a id="1">1. Rest Principals</a>

REST stands for <b>REpresentation State Transfer (REST)</b>. Any information that can be named can be a resource: a document or image, a temporal service (e.g. "today’s weather in Los Angeles"), a collection of other resources, a non-virtual object (e.g. a person), and so on.
 It comes within the Uniform Resource Identifiers <b>(URI)</b>. URI is the simplest element of the Web architecture and the most important.

### 1.1. Open API

- You <b>MUST define your REST contracts</b> using OpenAPI.
- Your <b>REST API SHOULD</b> be well documented using OpenAPI.
- <b>REST APIs MUST</b> be defined using the OpenAPI Specification.
- <b>API First Strategy</b>: APIs SHOULD be defined before coding starts
- REST APIs MAY be be defined using OpenAPI v3.0.0+ when the HATEOAS constraint is required.
- HATEOAS is an complex and expensive constraint to meet. It should not be the default choice for any API.
- Migration to OAS v3.0.0+ is planned for future phases of OpenAPI adoption.

### 1.2. Open API Levels

#### Richardson Model Maturity

Level | Name | Description
----- | ---- | -----------
0 | Swamp of POX | Use HTTP as a transport protocol (e.g., single endpoint which is POST) 
1 | Resources | An endpoint per resource type 
2 | HTTP Verbs | Use HTTP properties to deal with scalability and faults (e.g., HTTP methods, HTTP response codes)

- APIs SHOULD strive to meet Level 2: HTTP Verbs in the Richardson Maturity Model.
- APIs SHOULD NOT strive to meet Level 3: Hypermedia controls in the Richardson Maturity Model.


#### Entry level
The version of the operations/resources are specified as a integer in the basepath
- Scheme is set to https only 
- API Keys are only accepted as header parameters 
- The Finastra info object template has been implemented 
- consumes and produce are set to application/json at the top level 
- paths/resources are simple and exoteric instead of abstract and esoteric 
- The correct verbs are used for method calls 
- Applies the prescribed security practice 
- Request objects are validated

#### Advance level
- Meets all of the Entry level requirements 
- Conforms to the Finastra error handling standards 
- Swagger descriptions are used well to generate good human readable automatically generated docs. In other words copy for all of the descriptions are supplied or approved by Product Management or functional representative for the Product  # Full level 
- Meets all of the Advanced level requirements 
- Applies all of the Finastra approved data structures and naming conventions to the Swagger paths and definitions 
- usage of links and HOATEAS likes

#### Cutting edge
- Meets all of the Fully Compliant level requirements 
- Is up to date within 6 month of any additions to the Finastra Open API Standards 
- Good use of markdown in descriptions

## <a id="2">2. Naming conventions</a>

### 2.1. Resources
A key principle of REST involves separating your API into logical resources. These resources are manipulated using HTTP methods, and custom actions, which have semantic meaning for the resource.

Qualities of a resource:
- can be described by a noun 
- are concrete concepts 
- have a unique URL

<b>Use noun for resource Use plural for resource, that can return a collections Be consistent in plural singular. if a collection model exists it apply to all Use verb for action, but use classical HTTP verbs as much as possible.
</b>
```
paths:   
  /accounts:     
    "GET" < return a list     
    "POST" > create a single entity
```
#### Composite and path naming

- Resource <b>MUST not</b> contains char that requires URL encoding like space.
- <b>MUST follow</b> lower train case, also called kebab case, or lisp case, meaning hyphen ( - ) is the separator.

#### URI for dedicated action

In several use cases HTTP REST verb is not enough, because of the nature of the application, and the action we need to do on the resource. This can be called resource controller, basically it is an action that you set on top of a resource.

```
 POST 
    /accounts/{accountId}/transactions/{transid}/confirm
    , would confirm an existing transaction     
 but as same this would be     
 POST 
    /accounts/{accountId}/transactions/{transid}/confirmation, 
    that would create a confirmation
```

Stick to HTTP Verbs for CRUD operation. Actions and Commands SHOULD be a verb in present tense. Queries SHOULD be a pluralized noun.

#### Resource Identifier

Resource <b>MUST</b> be identified within the path, and not the path parameter.

<b>GET /accounts/{identifier}</b> MUST be the way to get a resource 
<b>GET /accounts?id={identifier}</b> can be used to retrieve a resource but in the semantic of doing a narrow search. and so should be avoided

<b>As a main rule you SHOULD not expose any logic inside the key/identifier.</b>

#### Child Resource

Use child resources when a resource logically contains other resources through aggregation.
```
/accounts/{id}/transactions
```

### 2.1. Resource Payload

- Attributes, parameter <b>MUST</b> be in lower <b>camelCase</b>.
- Child properties, <b>SHOULD not</b> repeat the parent key .

## <a id="3">3. HTTP Methods</a>

APIs MUST use the HTTP verbs as much as possible and follow ths semantic. APIs MUST NOT redefine the meaning of the standard HTTP methods. Handling single entity with {id} is a MUST Handling collection is highly recommended

HTTP | Verbs Action
---- | ------------
POST | Creation of a new resource
GET | Read-only; should not change any state
PUT | Full update/replace of the resource
PATCH | Partial update of a resource <b>(typically avoided)</b>
DELETE | Delete a resource

### 3.1. Get

The <b>GET</b> method is used to query an HTTP service. 

- <b> GET /customers/{id}</b> ⇒ return the customer identified with 123
- <b>GET /customers</b> ⇒ return list of customers. This SHOULD be implemented, and if makes sens with pagination, and search.

Characteristics:
- Request Payload : no
- Successful response payload: Yes
- Safe : Yes
- Idempotent : Yes
- Read-only
- Cacheable : Yes

Typical response status code:
- 200 OK
- 400 Bad Request
- 404 Not Found, if ID not found or invalid.

### 3.2. Post

The <b>POST</b> method is used to create new resources. Depending on the specific API context this can mean many things (e.g., creating a resource, executing an action, submitting long running operations, adding an item to a list). * Must inform the client how to access this new resource.

<b>POST /customers + payload</b> ⇒ return 201 ⇒ The industry practice is to return 201 response code with a location header ( optinal return the created resource)

Characteristics:
- Request Payload : Yes
- Successful response payload : Yes
- Safe :No
- Idempotent :No
- Informs the API consumer of the location of the new resource via Location header

Typical response status code:
- 200 (OK) bare minimum MUST
- 201 (Created) recommended but MAY
- 400 Bad Request
- 409 (Conflict) advice if resource already exists. OPTIONAL

<b>In opposition of GET , POST requests are never cached.</b>

### 3.3. Put

The <b>PUT</b> method is used to replace the state of an existing resource with the new state provided in the request. It is a full replacement.

<b>PUT /customers/123 + payload</b> with full content of the resource must be provided ⇒ return expected the full updated resource
<u><b>PUT /customers/ + payload</b> with a list ⇒ do a batch update</u>

Characteristics:
- Request Payload : Yes
- Successful response payload : No
- Safe : No
- Idempotent : Yes
- Cacheable : No

Typical response status code:
- 200 OK bare minimum MANDATORY
- 400 Bad Request
- 404 Not Found

### 3.4. Delete

The <b>DELETE</b> method is used to remove a resource. Depending on the context of the resource this can mean many things (e.g., remove the resource, remove an item from a list, cancel a long running operation).

<b>DELETE /customers/123</b> ⇒ return 204 , so no payload 
                             ⇒ return 200 , you may return the deleted object

Characteristics:
- Request Payload : commonly No
- Successful response Payload : No
- Safe :No
- Idempotent :Yes
- Cacheable : No

Note : When you invoke N similar DELETE requests, first request will delete the resource and response will be 200 (OK) or 204 (No Content). Other N-1 requests will return 404 (Not Found). Clearly, the response is different from first request, but there is no change of state for any resource on server side because original resource is already deleted. So, DELETE is idempotent.

Typical response status code:
- 200 OK
- 204 No content OPTIONAL
- 400 Bad Request
- 404 Not Found, if ID not found or invalid

### 3.5. Bulk Operations
- only implement bulk methods when necessary otherwise use series of simple call
- bulk operation SHOULD be atomic so when succeed all succeed, when fail all is rolledback
- consider how concurrency is controlled for bulk operations
- consider treating bulk operations as a long-running operation
- consider segregating bulk methods from the resource they operate upon
- consider how error handling will be reported back to the API consumer for each failed operation within the bulk operation
- consider whether a bulk upsert is feasible for simple bulk CRUD APIs

### 3.6. Other Actions

<b>Actions</b> are business capabilities which do not directly map to an existing HTTP method. These may be anything from a simple stateless function, without a resource representation, to complex workflows which interact with multiple resources.

#### Resource-less Actions
```
POST:
https://{host}/atm/transfer-funds
{
  "account-1": "...",
  "account-2": "...",
  "amount": "...",
...
}
```

#### Resource-based Actions
```
POST:
https://{host}/accounts/{id}/close-account
```

## <a id="4">4. HTTP Headers</a>

Header | Description
------ | -----------
Content-Type | Represents the format of the payload returned in the response. The first class candidate Content-type is application/json, as a content header in response to requests that return a HTTP body (all post and get requests)
Accept | Represents the format of the payload as input. The first class candidate and default value for server is application/json
Authorization | Allows Credentials to be provided to the Authorization / Resource Server depending on the type of resource being requested. For OAuth 2.0 / OIDC, this comprises of either the Basic / Bearer Authentication Schemes. Bearer token MUST follow the https://tools.ietf.org/html/rfc6750

## <a id="5">5. Error Codes</a>

### Error Codes

Response Code | Description
------------- | -----------
200 | <b>OK</b> Standard response for successful HTTP requests. The actual response will depend on the request method used. In a GET request, the response will contain an entity corresponding to the requested resource. In a POST request, the response will contain an entity describing or containing the result of the action.
201 | <b>Created</b> The request has been fulfilled, resulting in the creation of a new resource
204 | <b>No content</b> the server has successfully fulfilled the request and that there is no additional content to send in the response payload body.
304 | <b>Not Modified</b> RFC 7232 Indicates that the resource has not been modified since the version specified by the request headers If-Modified-Since or If-None-Match. In such case, there is no need to retransmit the resource since the client still has a previously downloaded copy.
400 | <b>Bad Request</b> The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing)
401 | <b>Unauthorized</b>must be used when there is a problem with the client’s credentials. A 401 error response indicates that the client tried to operate on a protected resource without providing the proper authorization. It may have provided the wrong credentials or none at all.
403 | <b>Forbidden</b> should be used to forbid access regardless of authorization state. A 403 error response indicates that the client’s request is formed correctly, but the REST API refuses to honor it. A 403 response is not a case of insufficient client credentials; that would be
401 | <b>(“Unauthorized”)</b> . REST APIs use 403 to enforce application-level permissions. For example, a client may be authorized to interact with some, but not all of a REST API’s resources. If the client attempts a resource interaction that is outside of its permitted scope, the REST API should respond with 403.
415 | <b>Unsupported Media Type</b>  status code indicates that the origin server is refusing to service the request because the payload is in a format not supported
404 | <b>Not Found</b>  must be used when a client’s URI cannot be mapped to a resource.The 404 error status code indicates that the REST API can’t map the client’s URI to a resource.
500 | <b>Internal Server Error.</b>  A generic error message, given when an unexpected condition was encountered and no more specific message is suitable

Conclusion :
- 200, 400, 401,404, 500 are Mandatory ( 200 OK / 400 client error / 500 server error )
- 201, 304,400,403 SHOULD BE used.
- Others like 204, 415, 503 MAY be used with parsimony
- APIs MUST NOT return 200 (OK) when there is an error or fault.

### RFC 7807
RFC Content can be found https://tools.ietf.org/html/rfc7807.

The RFC 7807 defined "Problem Details for HTTP APIs" , is as of today "proposed standard", and it SHOULD be applied. It proposes a seamless structure for common pattern of errors as same as an extension mechanism.

- "type" (string) - A URI reference [RFC3986] that identifies the problem type. This specification encourages that, when dereferenced, it provides human-readable documentation for the problem type (e.g., using HTML [W3C.REC-html5-20141028]). When this member is not present, its value is assumed to be "about:blank".
- "title" (string) - A short, human-readable summary of the problem type. It SHOULD NOT change from occurrence to occurrence of the problem, except for purposes of localization (e.g., using proactive content negotiation; see [RFC7231], Section 3.4).
- "status" (number) - The HTTP response status code ([RFC7231], Section 6). generated by the origin server for this occurrence of the problem.
- "detail" (string) - A human-readable explanation specific to this occurrence of the problem.
- "instance" (string) - A URI reference that identifies the specific occurrence of the problem. It may or may not yield further information if dereferenced. 

Here is a sample including as well a Zalando extension for chaining errors "cause":

```
{
  "type": "https://example.org/order-failed",
  "title": "Order failed",
  "status": 400,
  "cause": {
    "type": "https://example.org/out-of-stock",
    "title": "Out of Stock",
    "status": 400,
    "detail": "Item B00027Y5QG is no longer available"
  },
  "validationFailure" : [
    {
      "fields" : "email",
      "description" : "requires @"
    },
    {
      "fields" : "phone",
      "description" : "not valid "
    }
  ]
}
```

## <a id="6">6. Versioning</a>

For details on functional versioning please refere to the [Functional-Design-Guide.md](Functional-Design-Guide.md#apiversionning).

### Rest Versioning 

The most common versioning techniques in the industry today include:
- Path
- Query
- Custom headers
- Accept header with custom types

Version format
- Integer: 1, 2, 3
- Integer with prefix: v1, v2, v3
- Semantic with prefix: v1.0.0, v1.1.0, v2.0.0
- Date: 2017-12-25

#### Path parameter (URL-based)
Request Example:
```
GET https://myhostname/mydomain/{version}/{resource} HTTP/1.1
Accept: application+json
```
#### Query parameter
Request Example:
```
GET https://myhostname/mydomain/{resource}?v={version} HTTP/1.1
Accept: application+json
```
#### Custom headers
Request Example:
```
GET https://{host}/{resource} HTTP/1.1
Accept: application+json
X-API-Version: {version}
```
#### Accept header with custom types
Request Example:
```
GET https://{host}/{resource} HTTP/1.1
Accept: application/vnd.{product}.{version}+json
```
### Method choosed

- APIs MUST declare versions via <b>path parameter</b>. https://{host}/:version/{resource}
- APIs MUST NOT use semantic versioning.

The version we mention there is the one that have a direct impact on the client. It s based on the path. Swagger provides a field version, where here you can use the semantic versioning of your API, especially when you have new functionality. For instance:

- /v1/accounts + in swagger v1.0.0 for the initial version
- /v1/accounts + in swagger v1.0.1, is a micro change for instance adding documentation.
- /v1/accounts + in swagger v1.1.0, is a minor change for instance adding new functionality like adding end points.
- /v2/accounts + in swagger v2.0.0, is a major change, impact on the path itself, introduction of a breaking change.

## <a id="7">7. Navigation, Sorting, paging, filtering and seraching and projection</a>

Resource collections <b>SHOULD</b> support pagination

### Sorting 
- Resource collection APIs <b>SHOULD</b> have a sorting capability.
- Resource collection APIs <b>MUST</b> have a default sort clearly defined.
- If there is a limit in the maximum number of sort terms, it MUST be clearly defined.
- Use <b>sort</b> key word in query param, use desc for reverse order

```
By age in descending order and then name in ascending order.
GET /accounts?sort=age+desc,name+asc
```

### Paging

Offset / limit based pagination is more known than cursor-based pagination, so it has more framework support and is easier to use for API clients.

- The <b>limit</b> parameter controls the maximum number of items that may be returned for a single request. This parameter can be thought of as the page size.
- The <b>offset</b> parameter controls the starting point within the collection of resource results. For example, if you have a collection of 15 items to be retrieved from a resource and you specify limit=5, you can retrieve the entire set of results in 3 successive requests by varying the offset value: offset=0, offset=5, and offset=10.

Pagination <b>MUST</b> be done using the limit and offset pattern and keyword.

```
GET /accounts?limit=100 => return the 100 first accounts.
GET /accounts?offset=100 will return the accounts 101 and more
GET /accounts?offset=
```

### Paging output response

#### Solution 1 - Array - Highly not recommanded.
```
GET /accounts
[
{“id” : 1, “name” : “toto1”},
{“id” : 2, “name” : “toto2”}
]
```
#### Solution 2 - Collection embedded into a dedicated 'wellknown' attribute.
```
GET /orders?limit=3&offset=3
{
  "items" : [
  {
    "id": 4,
    "orderNumber": "1210",
    "status": "new"
  },
  {
    "id": 5,
    "orderNumber": "1100",
    "status": "delivered"
  }],
  "otherInfoOftheCollection" : "info like sum for instance"
  "_links": {
    "self": { "href": "/accounts" },
    "next": { "href": "/accounts?page=2" },
    "find": { "href": "/accounts{?id}", "templated": true }
  },
  "_meta" :
  {
    "timeToCompute" : 12,
    "inputofTheQuery" : "null"
  }
}
```
```
"_meta" :
{
  "limit" : 5 // number of item per page
  "pageCount" : 12 // number of pages
  "totalCount" : 63 // number of item
  "page" : 3 // current page
}
```
#### Solution 3 -  Json Hypertext Application Language (HAL)
HAL, is defined as currently a draft https://tools.ietf.org/html/draft-kelly-json-hal-08
Java HAL impelmentation can be found in https://github.com/mikekelly/hal_specification/wiki/Libraries#java

### Filtering & Searching

A convenient , and proven solution for search is using SQL, with operator like AND , OR. In the REST context usage or RSQL / FIQL syntax may be used. Details about those 2 standards are available. Still using RSQL a particular care MUST be done for SQL injection, and this even if behind the scene it is not a SQL engine.
- https://tools.ietf.org/html/draft-nottingham-atompub-fiql-00
- https://github.com/jirutka/rsql-parser

Filtering <b>SHOULD</b> be done with GET as query parameter this allows the search to be bookmarked.
<b>RSQL/FIQL</b> syntax <b>MAY</b> be used for the GET on the search resource.
```
GET /users?firstName=bob&lastname=tomak // basic solution using query parameter
GET /users/search?firstName=bob&lastname=tomak // basic solution using a dedicated endpoint
GET /users/search/mostActive // 'wellknown search'
GET /users/search/age=lt=5;age=gt=30 //using FIQL
```

### Projection, and dynamic content

When client knows that the payload can be heavy, and client is only interested with a subset of field a common way is to specify the lists of fields and sub fields he is interested in.

Example :
```
GET:
https://{host}/tickets?fields=subject,customerName,updatedAt
...
{
  "subject": "...",
  "customerName": "...",
  "updatedAt": "..."
}
```

## <a id="8">8. Asynchronous and long running operation</a>

Long running operations are treated like threads, or similar modern abstractions (e.g., Tasks, Futures). There are three common interfaces (listed in increasing levels of complexity):

1. Long Polling
2. Webhooks
3. Websockets

### Long Polling (Basic Solution)
Long polling is one of the easiest solution to put in place. the principle is the long running task return an id and the response status code 202 "The request has been accepted for processing, but the processing has not been completed". then the client pool for the new created resource and get a 202 till the action is completed.

```
Request (create task)
POST /complexReport HTTP/1.1
{...}

Response (return task Id) - Return the 202 Accepted HTTP response code.
HTTP/1.1 202 Accepted
Location: /complexReport/ef07186e-55d4-4db6-b160-f9afaa195d0a

Request (request task status by Id)
GET /complexReport/ef07186e-55d4-4db6-b160-f9afaa195d0a HTTP/1.1
Accept: application+json

Response (return status or redirect to task return value)
HTTP/1.1 202 Accepted
Location: /complexReport/ef07186e-55d4-4db6-b160-f9afaa195d0a
{
"status": "..."
}

Last
HTTP/1.1 200 OK
Location: /complexReport/ef07186e-55d4-4db6-b160-f9afaa195d0a
{
"reportData": [ ... ]
}

```

### Webhooks
Webhooks are HTTP callbacks that receive notification messages for events. we distinguish 2 kind of webhook the 1 to 1 model and the 1 to many.
<b>TO BE EXPLORED</b>

### Websockets
Websocket mechanism of upgrading protocol from HTTP to websocket is describe in the RFC6455. still there is no specification of the content it self of the message on the wire. In term of open API policies , websocket are not compliant with swagger. There is standards emerging to describe this like asyncAPI.

https://www.asyncapi.com/docs/getting-started/
## <a id="9">9. Caching</a>

consider Cache-Control header remember to use Last-Modified header remember to use ETag header (either strong or weak) consider supporting conditional requests consider enabling caching in Azure API Management, or your equivalent API Gateway.

APIs <b>MUST</b> prevent clients and intermediaries from caching sensitive information.

## <a id="10">10. Concurrency</a>

## <a id="11">11. Tokenization</a>
