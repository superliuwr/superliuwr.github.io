---
title: Token-Based Authentication System and JWT
date: 2018-06-09 16:50:13
tags: jwt
---

# Token-based authentication system
## How it works
1. Login with username and password
2. Obtain a token
3. Use the token to access restricted resources for a set time period. Usually put in headers: `Authorization: Bearer <token>`

# JWT
## What is JWT
JWT (pronounced 'jot') is a token based authentication system.

The **claims** in a JWT are encoded as a JSON object that is digitally signed using JSON Web Signature.

The JWT is a self-contained token which has authentication information, expire time information, and other user defined claims digitally signed.

<!-- more -->

## Old school way authentication with **sessions**
1. An object stored on the server that remembers if a user is still logged in, a reference to their profile, etc.
2. A cookie on the client-side that stores some kind of ID that can be referenced on the server against the session object's ID.

It doesn't work well with microservices when there are multiple backends and the session cookie we get from one backend won't correspond to another server.

## Advantages of JWTs 
1. No Session to Manage (**stateless**): The JWT is a self contained token which has authetication  information, expire time information, and other user defined claims digitally signed.
2. **Portable**: A single token can be used with multiple backends.
3. No Cookies Required, So It's Very **Mobile Friendly**
4. Good **Performance**: It reduces the network round trip time.
5. Decoupled/Decentralized: The token can be generated anywhere. **Authentication can happen on the resource server, or easily separated into its own server.**
6. Fine-grained access control. Within the token payload you can easily specify user roles and permissions as well as resources that the user can access.

## ANATOMY OF A JSON WEB TOKEN
A JSON Web Token consists of three parts: **Header**, **Payload** and **Signature**. The header and payload are **Base64 encoded**, then concatenated by a period, finally the result is algorithmically signed producing a token in the form of **header.payload.signature**. The header consists of metadata including the type of token and the hashing algorithm used to sign the token. The payload contains the claims data that the token is encoding.

### Header
The header typically consists of two parts: the type of the token, which is JWT, and the hashing algorithm such as HMAC SHA256 or RSA.
For example:

```javascript
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Then, this JSON is Base64Url encoded to form the first part of the JWT.

### Payload
The second part of the token is the payload, which contains the claims. 

Claims are statements about an entity (typically, the user) and additional metadata. 

There are three types of claims: reserved, public, and private claims.
* **Reserved claims**: These are a set of predefined claims, which are not mandatory but recommended, thought to provide a set of useful, interoperable claims. Some of them are: **iss** (issuer: Identifies principal that issued the JWT.), **exp** (expiration time), **sub** (subject: Identifies the subject of the JWT.), **aud** (audience: Identifies the recipients that the JWT is intended for. Each principal intended to process the JWT must identify itself with a value in the audience claim. If the principal processing the claim does not identify itself with a value in the aud claim when this claim is present, then the JWT must be rejected), among others.
Notice that the claim names are only three characters long as JWT is meant to be compact.
* **Public claims**: These can be defined at will by those using JWTs. But to avoid collisions they should be defined in the IANA JSON Web Token Registry or be defined as a URI that contains a collision resistant namespace.
* **Private claims**: These are the custom claims created to share information between parties that agree on using them.

An example of payload could be:
```javascript
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```
The payload is then Base64Url encoded to form the second part of the JWT.

### Signature
To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.

For example if you want to use the HMAC SHA256 algorithm, the signature will be created in the following way.
```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
The signature is used to verify that the sender of the JWT is who it says it is and to ensure that the message was’t changed in the way.

## Best Practices & Caveats
1. DO NOT store sensitive data like pasword anywhere in the token!
2. Give tokens an expiration
3. Embrace HTTPS. Do not send tokens over non-HTTPS connections as those requests can be intercepted and tokens compromised.  
4. httpOnly=true
5. Tokens stored in local/session storage can't be accessed from different domains (even if these are subdomains).
6. Tokens need to be stored somewhere (local/session storage or cookies).
7. Read [this article](https://auth0.com/blog/ten-things-you-should-know-about-tokens-and-cookies/)

## References
1. https://auth0.com/learn/json-web-tokens/
2. https://auth0.com/blog/ten-things-you-should-know-about-tokens-and-cookies/
3. https://scotch.io/tutorials/the-anatomy-of-a-json-web-token
4. https://en.wikipedia.org/wiki/JSON_Web_Token#Standard_fields
5. https://auth0.com/blog/authentication-in-golang/