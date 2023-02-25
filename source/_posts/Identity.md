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

JWTs are digitally signed.

They can be used to pass the identity of authenticated users between the identity provider and the service requesting the authentication.

They also can be authenticated and encrypted.

# Auth0

## Authenticate

