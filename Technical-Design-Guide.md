# Technical Design Guide

## 1. Rest Principals

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

## 2. Naming conventions

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

## 3. HTTP Methods

APIs MUST use the HTTP verbs as much as possible and follow ths semantic. APIs MUST NOT redefine the meaning of the standard HTTP methods. Handling single entity with {id} is a MUST Handling collection is highly recommended

HTTP | Verbs Action
---- | ------------
POST | Creation of a new resource
GET | Read-only; should not change any state
PUT | Full update/replace of the resource
PATCH | Partial update of a resource <b>(typically avoided)</b>
DELETE | Delete a resource



