---
title: GraphQL Authentication/Authorisation And Error Handling With Apollo
date: 2018-07-25 22:17:12
categories:
- API
tags:
- GraphQL
---

# Authentication/Authorisation

Where could we do access controll?

* on Express router not on apollo itself when running on HTTP
  * get `user/token` with an `auth` middleware before reaching `/graphql` endpoint
  * `user/token` will be put in the graphql context for later use(fine-grained authorisation etc.)
* on model layer
  * recommended by Facebook
  * good if you have a separate model layer API, for example a set of Restful APIs
* wrapper queries
  * access control and last-ditch error handling handled at resolver layer through chaining, similar to afterware and middleware
  * https://github.com/thebigredgeek/apollo-resolvers
* custom directives
  * good for limiting access to specific fields
  * https://www.youtube.com/watch?v=4_Bcw7BULC8
  * http://github.com/chenkie/graphql-auth
  * examples
    * directive @isAuthenticated on QUERY | FIELD
    * directive @hasScope(scope: [string]) on QUERY | FIELD

# Error Handling

How do we handle errors?
* errors handled in the standard errors array on the response body with a consistent machine-readable structure
* use error types defined by apollo-server, with custom names