# Technical Design Guide
Warning: This document is only copy/past notes from https://github.corp.dh.com/pages/ARPnP/api-design/ documents.

## Summary

- [1. Rest Principals](#1)
- [2. Naming conventions](#2)
- [3. HTTP Methods](#3)
- [4. HTTP Headers](#4)


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
