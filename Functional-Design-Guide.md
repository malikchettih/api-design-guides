# Functional Design Guide

When designing an API, pause for a second and think. What are the forces driving changes in my product.

## Open API Design

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

## API Check List

- High level description Comprehensive High level description message to facilitate the adoption
- Naming Conventions Compliance with the wording, usage of nouns, verbs , RPC style versus REST style , REST maturity level
- HTTP Verbs Accurate usage of the HTTP verbs POST / GET / DELETE / PUT
- HTTP Status Codes Accurate usage of HTTP response status code, compliance with RFC
- HTTP Headers All HTTP headers identified, including the standard ones
- Security concerns identified: access control, authentication, authorization.
- Documentation Static documentation, Behavioral documentation, Global description, Sample
- Mocking API mock that allow API consumer to build their client. Level complexity implemented

## API Versioning

There is still 2 notions of the versioning : 
- <b>public versioning</b> , will directly impact your client , we materialize it using path . Here we consider only breaking changes 
- <b>private versioning</b>, may impact your client, it is when you add end point for instance, change the description.

```
APIs MUST be versioned. Your API private versioning MUST be backwards compatible. if the private versioning introduce a breaking change. You MUST publish a new public version of your API when you are introducing breaking changes to the API contract.

APIs SHOULD differentiate finished API definitions from work in progress.

You MUST NOT deprecate any version which is still being used by
customers, without a notice period.
```
