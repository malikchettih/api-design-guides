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

## API Have CONTRACTS 

1. APIs are more than "sockets where I plug in a cable".
2. API contracts are scoped to a <b>Bounded Context</b>.



