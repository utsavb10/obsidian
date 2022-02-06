Refer to [page](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims)

[JSON web tokens (JWTs)](https://auth0.com/docs/tokens/concepts/jwts) claims are pieces of information asserted about a subject. For example, an [ID token](https://auth0.com/docs/secure/tokens/id-tokens) (which is always a JWT) can contain a claim called `name` that asserts that the name of the user authenticating is "John Doe". In a JWT, a claim appears as a name/value pair where the name is always a string and the value can be any JSON value. Generally, when we talk about a claim in the context of a JWT, we are referring to the name (or key). For example, the following JSON object contains three claims (`sub`, `name`, `admin`):

```json
    {
      "sub": "1234567890",
      "name": "John Doe",
      "admin": true
    }
    
```

There are two types of JWT claims:

-   **Reserved**: Claims defined by the [JWT specification](https://tools.ietf.org/html/rfc7519) to ensure interoperability with third-party, or external, applications. OIDC standard claims are reserved claims.
    
-   **Custom**: Claims that you define yourself. Name these claims carefully, such as through [namespacing](https://auth0.com/docs/secure/tokens/json-web-tokens/create-namespaced-custom-claims) (which Auth0 requires), to avoid collision with reserved claims or other custom claims. It can be challenging to deal with two claims of the same name that contain differing information.

## Reserved claims

The JWT specification defines seven reserved claims that are not required, but are recommended to allow interoperability with [third-party applications](https://auth0.com/docs/get-started/applications/confidential-and-public-applications/enable-third-party-applications). These are:

-   `iss` (issuer): Issuer of the JWT
    
-   `sub` (subject): Subject of the JWT (the user)
    
-   `aud` (audience): Recipient for which the JWT is intended
    
-   `exp` (expiration time): Time after which the JWT expires
    
-   `nbf` (not before time): Time before which the JWT must not be accepted for processing
    
-   `iat` (issued at time): Time at which the JWT was issued; can be used to determine age of the JWT
    
-   `jti` (JWT ID): Unique identifier; can be used to prevent the JWT from being replayed (allows a token to be used only once)
    

You can see a full list of reserved claims at the [IANA JSON Web Token Claims Registry](https://www.iana.org/assignments/jwt/jwt.xhtml#claims).

## Custom claims

You can define your own custom claims which you control and you can add them to a token using a rule. You must use collision-resistant name using [namespacing](https://auth0.com/docs/tokens/guides/create-namespaced-custom-claims) (which Auth0 requires).

Here are some examples:

-   Add a user's email address to an access token and use that to uniquely identify the user.
    
-   Add custom information stored in an Auth0 user profile to an ID token.
    

As long as your rule is in place, the custom claims it adds will appear in new tokens issued when using a [refresh token](https://auth0.com/docs/secure/tokens/refresh-tokens).

For an example showing how to add custom claims to a token, see [Sample Use Cases: Scopes and Claims](https://auth0.com/docs/get-started/apis/scopes/sample-use-cases-scopes-and-claims).

### Public claims

You can create custom claims for public consumption, which might contain generic information like name and email. If you create public claims, you must either register them or use collision-resistant names through namespacing (which Auth0 requires) and take reasonable precautions to make sure you are in control of the namespace you use.

In the [IANA JSON Web Token Claims Registry](https://www.iana.org/assignments/jwt/jwt.xhtml#claims), you can see some examples of public claims registered by OpenID Connect (OIDC):

-   `auth_time`
    
-   `acr`
    
-   `nonce`
    

### Private claims

You can create private custom claims to share information specific to your application. For example, while a public claim might contain generic information like name and email, private claims would be more specific, such as employee ID and department name.

According to the JWT standard, you should name private claims cautiously to avoid collision, such as through namespacing (which Auth0 still requires). Private claims should not share names with reserved or public claims.