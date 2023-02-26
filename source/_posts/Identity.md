---
title: Identity & Auth0
date: 2023-02-22 15:57:10
categories:
  - [全栈]
---

From [Auth0 Docs](https://auth0.com/docs/get-started/identity-fundamentals). [Glossary](https://auth0.com/docs/glossary)

<!-- more -->

# Identity Fundamentals

**Identity and Access Management (IAM)** provides control over **user validation** and **resource access**.

IAM ensures that the **right people** access the **right digital resource** at the **right time** and for the **right reason**.

IAM usually combines these capabilities:

- Seamless signup and login experiences
- Multiple source of user identities
- Multi-factor authentication (MFA)
- Step-up authentication
- Attack protection
- Role-based access control (RBAC)

## Basic Concepts

- **Digital resource**: Combination of applications and data, e.g. web applications, APIs, platforms, devices, or databases.
- The core of IAM is **identity**.
  - In IAM, a **user account** is a **digital identity** (Set of attributes that define a particular user).
  - User account can also represent non-humans, such as software, IoT devices, or robotics.

### Authentication & Authorization

| **Authentication**                                           | **Authorization**                                            |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| Determines whether users are who they claim to be            | Determines what users can and cannot access                  |
| Challenges the user to validate credentials (for example, through passwords, answers to security questions, or facial recognition) | Verifies whether access is allowed through policies and rules |
| Usually done before authorization                            | Usually done after successful authentication                 |
| Generally, transmits info through an ID Token                | Generally, transmits info through an Access Token            |
| Generally governed by the OpenID Connect (OIDC) protocol     | Generally governed by the OAuth 2.0 framework                |
| Example: Employees in a company are required to authenticate through the network before accessing their company email | Example: After an employee successfully authenticates, the system determines what information the employees are allowed to access |

### Identity Providers

An identity provider creates, maintains, and manages identity information, and can provide authentication services to other applications. E.g., Google Account, Facebook, LinkedIn, Microsoft Active Directory and Swedish BankID.

Identity Providers don't share your authentication credentials with the app that rely on them.

### Authentication Factors

| Factor type                     | Examples                                   |
| :------------------------------ | :----------------------------------------- |
| Knowledge (something you know)  | Pin, password                              |
| Possession (something you have) | Mobile phone, encryption key device        |
| Inherence (something you are)   | Fingerprint, facial recognition, iris scan |

## Protocols

Some specifications and protocols that provide guidance on how to:

- Design IAM systems to manage identity
- Move personal data securely
- Decide who can access resources

These IAM industry standards are considered to the most secure, reliable, and practical to implement:

### OAuth 2.0 Protocol

Lets an app access resources hosted by other web apps on behalf of a user without ever sharing the user's credentials.

It allows third-party developers to rely on large social platforms like Facebook, Google and Twitter for login.

- **Resource Owner**: Entity that can grant access to a protected resource. (**end-user**).
- **Client**: **Application** requesting access to protected resource on behalf of the **Resource Owner**.
- **Resource Server**: Server hosting the protected resource. (**the API** you want to access).
- **Authorization Server**: Server that authenticates the **Resource Owner** and issues **Access Tokens** after getting proper authorization (e.g. Auth0).
- **User Agent**: Agent used by **Resource Owner** to interact with the Client (e.g. a browser or a native application).

#### Grant Types

OAuth 2.0 defines flows to get an access token, these flows are called grant types.

##### Client Credentials Flow

- Machine-to-machine (M2M) applications (CLIs, daemons, services running on back-end).
- No username & password.
- Pass **Client ID** and **Client Secret** to authenticate and get a token.

![Client Credentials Flow](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Client_Credentials_flow.png)

##### Authorization Code Flow

- Server-side web applications
- Exchange an **Authorization Code** for a token, also pass along **Client Secret**

![Authorization Code Flow](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/auth-sequence-auth-code.png)

In short:

1. **The user** click login on the **web app**, and the web app request authorization from **Authorization Server**.
2. **Authorization Server** asks the **user** for authorization and get **Authorization Code**. And redirect to the **web app**.
3. Server-side web application gets the **Authorization Code**, and send it to **Authorization Server** with **Client Secret**, to get the token.

##### Resource Owner Password Flow

- Must not be used by third-party clients. Only highly-trusted applications can use this flow.
- Requests users provide credentials (username and password), typically using an interactive form.
- Even if the application is highly-trusted, this flow should only be used when redirect-based flows (like Authorization Code Flow) cannot be used.

![Resource Owner Password](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ROP_Grant.png)

##### Authorization Code Flow with Proof Key for Code Exchange (PKCE)

- **Public Clients** (Native and Single-Page Applications).

![Authorization Code Flow with PKCE](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/auth-sequence-auth-code-pkce.png)

In short:

1. There is no **Client ID** and **Client Secret** stored in the app.
2. The **app** generates cryptographically-random **Code Verifier**, and from this generates **Code Challenge**.
3. **Authorization Server** stores **Code Chanllenge** and issue **Authorization Code** like normal Authorization Code Flow.
4. The **app** send the **Authorization Code** and **Code Verifier** to **Authorization Server**, where the **Code Verifier** and **Code Challenge** will be validated.

##### Implicit Flow with Form Post

- No **Access Token** (like JWT, to access some API), only ID Token (just for the client itself).
- For front-end centered applications (maybe there are no backend)
- Implicit Flow can also not be done with form post, it can also get token through redirected fragment.

![Implicit Flow with Form Post](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/auth-sequence-implicit-form-post.png)

##### Device Authorization Flow

- Input-constrained devices asks the user to go to a link on their computer or smartphone and authorize the device.

![Device Authorization Flow](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/auth-sequence-device-auth.png)

#### Endpoints

##### Authorization Endpoint

> `/authorize`

- Interact with **resource owner** and **get the authorization** to access the protected resource.
- E.g. 3rd party app uses Google Account to login.
  - First, the app will redirect to Google to **authenticate**.
  - Then you will get a consent screen, where you asked to **authorize** the app to access some of your data (email address and contacts, etc.).

| Parameter       | Description                                                  |
| :-------------- | :----------------------------------------------------------- |
| `response_type` | Tells the authorization server which grant to execute.       |
| `response_mode` | (Optional) How the result of the authorization request is formatted. Values:<br />- `query`: for Authorization Code grant. `302 Found` triggers redirect.<br />- `fragment`: for Implicit grant. `302 Found` triggers redirect.<br />- `form_post`: `200 OK` with response parameters embedded in an HTML form as hidden parameters.<br />- `web_message`: For Silent Authentication. Uses HTML5 web messaging. |
| `client_id`     | The ID of the application that asks for authorization.       |
| `redirect_uri`  | Holds a URL. A successful response from this endpoint results in a redirect to this URL. |
| `scope`         | A space-delimited list of permissions that the application requires. |
| `state`         | An opaque value, used for security purposes. If this request parameter is set in the request, then it is returned to the application as part of the `redirect_uri`. |
| `connection`    | Specifies the connection type for Passwordless connections   |

##### Token Endpoint

> `/oauth/token`

- Used by the **application** to get an **Access Token** or a **Refresh Token**.
- **Implicit Flow**: not used. Because an access token is issued directly through authorization endpoint.
- **Authorization Code Flow**: The app exchanges **authorization code** for access token.
- **Client Credentials Flow, Resource Owner Password Flow**: The app authenticates using a set of credentials and get an access token. 

#### State Parameters

- Allows you to restore the previous state of your application.
- To mitigate CSRF attacks. 

### Open ID Connect (OIDC) Protocol

- A simple identity layer that sits on the top of OAuth 2.0.

  - While OAuth 2.0 is about resource access and sharing, OIDC is about user authentication.

  - OIDC makes it easy to verify a user's identity and obtain basic profile information from the identity provider.

- Uses **JSON Web Tokens (JWTs)**.
  - JWTs contain **claims** (statements, such as name or email address) about an entity and additional metadata.
  - OpenID Connect Specification defines a set of standard claims, including name, email, gender, and so on.
  - You can also create custom claims and add them to your tokens.

### Security Assertion Markup Language (SAML)

SAML is an open-standard, XML-based data format for authentication and authorization between two entities without a password:

- **Service Provider** (SP): Agrees to trust the identity provider to authenticate users.
- **Identity Provider** (IP): Authenticates users and provides to SP an authentication assertion that indicates a user has been authenticated.

### Web Services Federation (WS-Fed)

Developed by Microsoft. It defines the way security tokens can be transported between different entities to exchange identity and authorization information.

## Tokens

### JSON Web Tokens (JWTs)

- JSON Web Token (JWT), pronounced "jot", is an open standard that defines a compact and self-contained way for securely transmitting information between parties as a JSON object.
- JWT is digital signed, and can also be encrypted.

#### Introduction

##### Benefits

> Compared to **Simple Web Tokens (SWTs)** and **Security Assertion Markup Language (SAML)** tokens.

- **More compact**: It is smaller than SAML token. 
- **More secure**
  - Can use public/private key pair in the form of an X.509 certificate for signing. (In comparison, signing XML with XML Digital Signature without introducing obscure security holes is very difficult.)
  - Can be symmetrically signed by a shared secret using the HMAC algorithm.

- **More common**: JSON parsers are common than XML parsers in most programming languages.
- **Easier to process**: JWT is used at internet scale. And can be sent through a RUL, through a POST parameter, or inside an HTTP header, and it is transmitted quickly.

##### Use

- **Authentication**: According to the OpenID Connect (OIDC) specs, an **ID Token** is always a JWT.
- **Authorization**: An **Access Token** may be a JWT. Single Sign-on (SSO) widely uses JWT.
- **Information Exchange**

#### Structure

All Auth0-issued JWTs have **JSON Web Signatures (JWTs)**, meaning they are **signed** rather than **encrypted**.

- A JWS represents content secured with digital signatures or Message Authentication Codes (MACs) using JSON-based structures.

A well-formed JWT consists of three concatenated Base64url-encoded strings, separated by dots (`.`):

```text
JOSE Header.JWS Payload.JWS Signatrue

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

[JWT.io Debugger](https://jwt.io/?_gl=1*36goab*rollup_ga*MTE1MjA5ODQwOC4xNjc3MDQ2ODg4*rollup_ga_F1G3E656YZ*MTY3NzM5NjU5Ni4xMi4xLjE2NzczOTg2NzUuNC4wLjA.&_ga=2.71229206.781441431.1677243082-1152098408.1677046888) allows you to quickly check that a JWT is well formed and to manually inspect the values of the various claims.

##### JOSE Header

JOSE (JSON Object Signing and Encryption) Header is comprised of:

- `alg`: The hashing algorithm being used (e.g., HMAC SHA256 or RSA).
- `typ`: The type of the JWT.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

##### JWS Payload

The payload contains statements about the entity, and additional entity attributes, which are called claims.

```json
{
  "sub": "123456789",
  "name": "John Doe",
  "admin": true
}
```

##### JWS Signature

Is used to **verify that the sender** of the JWT is who it says, and **ensure that the message wasn't changed** along the way.

```text
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

#### Claims

JWT claims are pieces information asserted about a subject.

E.g. The following JSON object contains three claims (`sub`, `name`, `admin`):

```json
{
  "sub": "123456789",
  "name": "John Doe",
  "admin": true
}
```

There are 2 types of JWT claims:

- **Registered**: Standard claims registered with the [IANA](https://www.iana.org/assignments/jwt/jwt.xhtml) and defined by the [JWT Specification](https://www.rfc-editor.org/rfc/rfc7519).
- **Custom**: Consists of non-registered public or private claims. Public claims are collision-resistant while private claims are subject to possible collisions.

##### Registered Claims

- `iss` (issuer): Issuer of the JWT
- `sub` (subject): Subject of the JWT (the user)
- `aud` (audience): Recipient for which the JWT is intended
- `exp` (expiration time): Time after which the JWT expires
- `nbf` (not before time): Time before which the JWT must not be accepted for processing
- `iat` (issued at time): Time at which the JWT was issued; can be used to determine age of the JWT
- `jti` (JWT ID): Unique identifier; can be used to prevent the JWT from being replayed (allows a token to be used only once)

##### Custom Claims

###### Public Claims

If you create public claims, you must either register them or use collision-resistant names through namespacing and take reasonable precautions to make sure you are in control of the namespace you use.

In the [IANA JSON Web Token Claims Registry](https://www.iana.org/assignments/jwt/jwt.xhtml#claims), you can see some examples of public claims registered by OpenID Connect:

- `auth_time`
- `acr`
- `nonce`

###### Private Claims

You can create private custom claims to share information specific to your application.

#### Signing Algorithms

- **RS256** (RSA Signature with SHA-256): An asymmetric algorithm.
- **HS256** (HMAC with SHA-256): A symmetric algorithm.

RS256 is recommended, because:

- Only the holder of private key can sign tokens, while anyone can check if the token is valid using the public key.
- Can request a token that is valid for multiple audiences.
- If the private key is compromised, you can implement key rotation without having to re-deploy your application or API with the new secret. (Which you have to do if using HS256)

### ID Tokens

- Cache user profile information and provide it to a client application.
- The application receives an ID token after a user successfully authenticates, then consumes the ID token and extracts user information from it.
- **ID Token should never be used to obtain direct access to APIs to make authorization decisions.**
- By default, an ID token is valid for 10 hours.

#### Validate ID Tokens

1. Validate the JWT.
2. Check additional standard claims, including:
   1. **Token audience** (`aud`, string): The audience value for the token must match the client ID of the application.
   2. **Nonce** (`nonce`, string): Passing a nonce in the token request is recommended (required for the Implicit Flow) to help prevent replay attacks. The nonce value in the token must exactly match the original nonce sent in the request.

### Access Tokens

- Allow an application to access an API.
- If you have chosen to allow users to login through an **Identity Provider (IdP)**, such as Facebook, the IdP  will issue its own access token to allow your application to call the IDP's API.

#### Validate Access Tokens

An Access Token is meant for an API and should be validated only by the API for which it was intended.

If any of these checks fail, the token is considered invalid, and the request must be rejected with `401 Unauthorized` result.

1. Validate the JWT.

2. Verify token audience claims. The `aud` filed could contain both an audience corresponding to your custom API and an audience corresponding to the `/userinfo` endpoint.

3. Verify permissions (scopes). Check the `scope` claim. It should match the permissions required for the endpoint being accessed. E.g.

   - `create:users` provides access to the `/create` endpoint

   - `read:users` provides access to the `/read` endpoint

### Refresh Tokens

- An **OAuth Refresh Token** is a credential artifact that OAuth can use to get a new Access Token without user interaction.
- This allows the Authorization Server to shorten the access token lifetime. You can request new access tokens until the refresh token is on the DenyList.
- **It is important to keep the number of refresh tokens within a reasonable manageable limit.** Applications must store refresh tokens securely because they essentially allow a user to remain authenticated forever.

### Token Best Practices

#### Basics to Keep in Mind

- **Keep it secret. Keep it safe.** The signing key should be treated like any other credential and revealed only to services that need it.
- **Do not add sensitive data to the payload.** Tokens are signed to protect against manipulation and are easily decoded.
- **Give tokens an expiration.**
- **Embrace HTTPS.** Do not send tokens over non-HTTPS connections as those requests can be intercepted and tokens compromised.
- **Consider all of your authorization use cases.**
- **Store and reuse.**

# Auth0

## Authenticate

