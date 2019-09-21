---
layout: post
title: API Best Practices
---

REST represents an entity.

## Commands: 

GET/PATCH/PUT/DELETE 

*** Idempotency 

## URL

- api location
  - domain: `https://api.example.com`
  - path: `https://example.com/api`
- [version](https://gist.github.com/ron23/22137095cd3f04cd89f6c75d313202af#versioning)
- entity (only nouns)
  - Don't use the command in the name (example: don’t use `/getUser`, use GET /user)
  - To pull specific entity: `/api/v1/orders/6`
  - can even go deeper: `/api/v1/order/6/items` => all the items of order 6
  - add query logic: `/api/v1/order/6/items?contains=abc`
- Rule of thumb: if the value identify the entity it goes in the path. 
- singular vs plural: no concensus. Ideally make it readable: `/orders` returns a list and `/order` return an item. 
from a client side prespective, I prefer `/orderes` and always return a list.

## Response codes:
RFC7807 defines a standard for returning erros.

https://blog.restcase.com/rest-api-error-handling-problem-details-response/

2XX - success

3XX - redirects

4XX - Tell the user what's wrong. RFC7807. Can contain code, message, etc.

5XX - Always return `internal server error`. User (browser/other service) shouldn't know anything.

What to do when some operations are successful but some failures?
Return 200 and return in the body what failed/succeeded

## Versioning: 
- Consider default version if not specified (latest/fail/first)
- Where?
  - HTTP Header. Harder to implement, less visibile.
  - Path: `GET /api/v1`. Most popular.
  - Query string: `GET api?v=1` and with POST in the body. Not very RESTful.
  
## Documentations: 
Use swagger. No questions asked. Takes minutes to set up. Write the API first.

## Consistency
- Stay consistent across teams
- Don't mix queryString and path: `users?id=123` vs `users/123`

## Authorization & Authentication
- Authorization - who you are
- Authentication - what can you do

Use JWT. JWT contains the claims: groups/roles etc.

OAuth: FB/Google etc provide the JWT encrypted. During the app registration process the OAuth provider provides the public key to decrypt the token.

Pass Token via Authentication Header `Bearer <jwt>`

## Monitoring and Analytics
- New Relic
- AWS Cloudwatch

## API Gateway / Discovery
In a micro-services system, when a service needs to talk to another service he can either:
- **API Gateway**: Infront of service Single point of contact. It points the request to the right place.
- **Discovery**: like a phone book / DNS. The microservice will ask the discovery service for the right url.

API Gateway can also give:
  - Authorization / Authntication
  - Quota limit - bussiness decision usually
  - Rate limit - used to handle load (max requests per X) 
  - publish/unpublish/deprecated API
  - versioning
  - logging / monitoring
  - SSL offloading 
 
Problems with APIG:
  - Latency (network hop)
  - Cost
  - Adds complexity / points of failure
  - Dependecy on cloud

## HTTP vs Socket
- with socket load balancing becomes harder

## Problems with REST
- not built for actions. For example to validate something - it's not RESTFUL
- PUT / PATCH is confusing
 
## Interesting links:
- https://blog.restcase.com/rest-api-error-handling-problem-details-response/
- https://www.ranlevi.com/2019/04/02/osim_software_api/
- https://www.ranlevi.com/2019/07/09/osim_software_api-2/ 
