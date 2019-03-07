# Functional Design Guide
Warning: This document is only copy/past notes from https://github.corp.dh.com/pages/ARPnP/api-design/ documents.

When designing an API, pause for a second and think. What are the forces driving changes in my product.

## 1. Open API Design

When API model is designed, it has to be:
### Be "Semantically Right", not just "Syntactically Right".

1. APIs must be reusable.
2. Materialized arround Bounded Context.
3. API should be minimal and complete
4. Manage the API Evolution
5. Encapsulate (again!)

- The API model should be scoped to exists in  single bounded context. DDD calls that the ubiquituous language.
- API should be complete and kept simple, ideally doing just 1 atomic thing very well.

### API Have CONTRACTS 

1. APIs are more than "sockets where I plug in a cable".
2. API contracts are scoped to a <b>Bounded Context</b>.

## 2. API Check List

- High level description Comprehensive High level description message to facilitate the adoption
- Naming Conventions Compliance with the wording, usage of nouns, verbs , RPC style versus REST style , REST maturity level
- HTTP Verbs Accurate usage of the HTTP verbs POST / GET / DELETE / PUT
- HTTP Status Codes Accurate usage of HTTP response status code, compliance with RFC
- HTTP Headers All HTTP headers identified, including the standard ones
- Security concerns identified: access control, authentication, authorization.
- Documentation Static documentation, Behavioral documentation, Global description, Sample
- Mocking API mock that allow API consumer to build their client. Level complexity implemented

## 3. API Versioning

There is still 2 notions of the versioning : 
- <b>public versioning</b> , will directly impact your client , we materialize it using path . Here we consider only breaking changes 
- <b>private versioning</b>, may impact your client, it is when you add end point for instance, change the description.

APIs <b>MUST be versioned</b>. Your API private versioning MUST be backwards compatible. if the private versioning introduce a breaking change. You MUST publish a new public version of your API when you are introducing breaking changes to the API contract.
APIs SHOULD differentiate finished API definitions from work in progress. You MUST NOT deprecate any version which is still being used by customers, without a notice period.

### Backward compatibility

#### Backward compatible , Non Breaking change
- Adding an endpoint to an existing API , extending resource end point , adding a verb.
- Extending the range of a request value.As soon as the parameter is not a mandatory one , adding a parameter in any part query, path,
body) of the request is safe.
- Adding a field to a response message (not a resource )
- Adding a value to an enum
- Behavioural change, As soon as it is planned by the API is not considered as a breaking change.

#### Backwards-incompatible (breaking) changes
- Removing or renaming a path, field, enum value.
- Changing an HTTP binding.
- Changing the type of a field.
- Adding a read/write field to a resource message

## 4. Localization & Internationalization

Out of scope for APIs:
• Localization
• Translations
• RTL and other orientations
• Internationalization
• Decimal separators
• Currency

In scope for APIs:
• Date
• Date-Time
• Time

APIs which support either Localization or Internationalization MUST understand the <b>Accept-Language</b> header to facilitate content negotiation with the client and inform the client of the selection in the <b>Content-Language</b> header.

The standard HTTP Header <b>Accept-Language</b> informs the service which languages the client is able to understand, and which locale variant is preferred. Using content negotiation, the service selects one of the proposed language and locales, and confirms the selection within the response header.

Content-Language. Browsers use this header to align their UI.

The Accept-Language header format contains two items: language, culture.

In the following example, the French language is identified with the two letter code <b>FR</b>; the Swiss culture/locale, which is optional in this format, is identified with the two letter code <b>ch</b>.

Example
```
FR-ch
```

## 5. Data Standards

APIs <b>MUST</b> support JSON format.

The API client <b>MUST NOT</b> rely on the ordering of properties or values within the JSON payload.

APIs must use ISO 8601 representations of both Date and DateTime types in JSON payloads.

APIs <b>SHOULD NOT</b> use TimeStamp types within JSON payloads.

APIs <b>SHOULD</b> define a dedicated content type when transferring binary as base64 encoded text.

## 6. Tokenization

APIs <b>MAY</b> protect against enumeration attacks. Tokenization must be applied to share public content. Token represents the secret.

```
GET /resources/3
GET /resources/4
GET /resources/5
...
GET /resources/10
```
Tokenization is one solution to this problem. Instead of returning the <b>integer id</b>, return a <b>uuid</b> id which has been permanently mapped to the original id.

```
GET /resources/11755216-8bbd-423f-a596-373d66f8840b
GET /resources/2f51533d-ace4-4801-9adb-2bca259f7852
GET /resources/6f651922-d5c5-4fba-a1c5-ba62a069bc17
...
GET /resources/53d58cd7-17bd-4e49-b04f-a7082ea40dd2
```

The tokenization capability can be added onto legacy services as a proxy, or as a plug-in at the API gateway.
