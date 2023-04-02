---
layout: post
title: 🗂 【Daily】Question with answer - 2023
date: 2023-03-04 00:00
---

## Summary
This post contains some short answers of the technology questions I met in my daily work in 2023, including some other LINK for further details answers. Will keep updating this post for a year.

### What are the popular ways to maintain a user’s authentication state nowadays?  

Http is stateless protocol, which means for every separate request, it must contain information to tell the server who you are. There are mainly two ways to do it, **session token** and **JWTs**. 

![Request Authentication](https://typora-1302119905.cos.ap-nanjing.myqcloud.com/Coding/RequestAuth.png)

---

### How does sessionID or session token work?

Every time when user sends first request to try to login to a serve, serve side will validate the data of username or password. Once success, server side will generate a sessionID and store it, then return a cookie with the sessionID. Then for the following request to this serve will always include that cookie.

For every request with cookie, Serve could identify which user is sending this request base on the sessionID of the cookie. Some cons for session based authentication is it would increase the storage cost in the serve side as more user login, cause serve need to store the mapping between user and sessionID in it.  

---

### What is JWT?

JWT(Json Web Token) is a token sent by serve to client when login. It contains 3 sections, which are header, payload and signature (signature of the header and payload). In the payload, it normally contains information like user role, user id, issuer (the serve where the jwt token from). when client use this jwt token to request the serve, serve could know who is sending the request. Compare to sessionID, which need the serve to store the mapping of sessionID and user id, using jwt contains the user id and serve could directly get it. 

--- 

### Pros & Cons of SessionID and JWT for Authentication

#### SessionID

Pros:
1. Can logout the user from serve side (Revoking user's access immediately)
2. Session data is not visitable (like user id, role)

Cons:
1. It will introduce notable latency to the response time from the serve. For every request serve need to verify the sessionID and figure out the associate user id for it, either use cache or accessing db for it
2. While the amount of login user increases, more and more sessionID generated and store to cache or db. It needs more cost to maintain session storage. 

#### JWT

Pros:
1. Reduce the latency since serve side could do the authentication and figure out who is sending this request quicker
2. Provider more scalability for increasing users. JWT is only stored in client side. Serve doesn't need to store the mapping of sessionID and userID

Cons:
1. By using JWT, serve could not make a user logout and revoke role or privileges easily. JWT token will keep in valid status before the expired time pass.
2. Session data is visitable. JWT token is only base-64 url encoded content, could easily decode it and get the information on it.

- [JSON Web tokens vs sessions for authentication, should you use JWTs as session tokens?](https://www.youtube.com/watch?v=U6OcC0yq1CE)
- [Session Tokens Vs. JWTs: Choosing Your Session Management Solution](https://devops.com/session-tokens-vs-jwts-choosing-your-session-management-solution/)
- [JWTs vs. sessions: which authentication approach is right for you?￼](https://stytch.com/blog/jwts-vs-sessions-which-is-right-for-you/)
