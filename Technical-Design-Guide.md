# Technical Design Guide

## 1. Rest Principals

REST stands for <b>REpresentation State Transfer (REST)</b>. Any information that can be named can be a resource: a document or image, a temporal service (e.g. "today’s weather in Los Angeles"), a collection of other resources, a non-virtual object (e.g. a person), and so on.
 It comes within the Uniform Resource Identifiers <b>(URI)</b>. URI is the simplest element of the Web architecture and the most important.

### Open API

- You <b>MUST define your REST contracts</b> using OpenAPI.
- Your <b>REST API SHOULD</b> be well documented using OpenAPI.
- <b>REST APIs MUST</b> be defined using the OpenAPI Specification.
- <b>API First Strategy</b>: APIs SHOULD be defined before coding starts
- REST APIs MAY be be defined using OpenAPI v3.0.0+ when the HATEOAS constraint is required.
- HATEOAS is an complex and expensive constraint to meet. It should not be the default choice for any API.
- Migration to OAS v3.0.0+ is planned for future phases of OpenAPI adoption.

#### Richardson Model Maturity

Level | Name | Description
----- | ---- | -----------
0 | Swamp of POX | Use HTTP as a transport protocol (e.g., single endpoint which is POST) 
1 | Resources | An endpoint per resource type 
2 | HTTP Verbs | Use HTTP properties to deal with scalability and faults (e.g., HTTP methods, HTTP response codes)

- APIs SHOULD strive to meet Level 2: HTTP Verbs in the Richardson Maturity Model.
- APIs SHOULD NOT strive to meet Level 3: Hypermedia controls in the Richardson Maturity Model.

### Open API Levels

#### 1. Entry level
The version of the operations/resources are specified as a integer in the basepath
- Scheme is set to https only 
- API Keys are only accepted as header parameters 
- The Finastra info object template has been implemented 
- consumes and produce are set to application/json at the top level 
- paths/resources are simple and exoteric instead of abstract and esoteric 
- The correct verbs are used for method calls 
- Applies the prescribed security practice 
- Request objects are validated

#### 2. Advance level
- Meets all of the Entry level requirements 
- Conforms to the Finastra error handling standards 
- Swagger descriptions are used well to generate good human readable automatically generated docs. In other words copy for all of the descriptions are supplied or approved by Product Management or functional representative for the Product  # Full level 
- Meets all of the Advanced level requirements 
- Applies all of the Finastra approved data structures and naming conventions to the Swagger paths and definitions 
- usage of links and HOATEAS likes

#### 3. Cutting edge
- Meets all of the Fully Compliant level requirements 
- Is up to date within 6 month of any additions to the Finastra Open API Standards 
- Good use of markdown in descriptions

