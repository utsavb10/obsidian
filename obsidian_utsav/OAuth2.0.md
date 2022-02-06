###### About OAuth and OpenId
Plain language understanding of OAuth -- [youtube](https://www.youtube.com/watch?v=996OiexHze0&t=2972s)

###### Reference Doc
[# Identity, Claims, & Tokens – An OpenID Connect Primer, Part 1 of 3](https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1#:~:text=There%20are%20three%20types%20of,%3A%20id_token%20%2C%20access_token%20and%20refresh_token%20.)

OIDC means OpenId

Scope -- [Definition](https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1#whats-a-scope)
Claims -- [Definition](https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1#whats-a-claim)
[[information_on_particular_claims]]

**Access tokens** are used as bearer tokens. A **bearer token** means that the bearer can access authorized resources without further identification.

###### [Identifying Token Types](https://developer.okta.com/blog/2017/07/25/oidc-primer-part-1#identifying-token-types)

It can be confusing sometimes to distinguish between the different token types. Here’s a quick reference:

1.  ID tokens carry identity information encoded in the token itself, which must be a JWT
2.  Access tokens are used to gain access to resources by using them as bearer tokens
3.  Refresh tokens exist solely to get more access tokens


[[oauth_implementations]] on Resource Server implementation, role based access.