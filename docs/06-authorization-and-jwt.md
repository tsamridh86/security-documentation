# Authorization & JSON Web Tokens (JWT)

> **Authentication (AuthN)** is establishing *who* you are.
> **Authorization (AuthZ)** is establishing *what* you can do.

Up until now, we've mostly discussed encryption and trust (Authentication). Now, let's look at how we manage permissions. But before we dive into technical details, let's first understand statelessness and statefulness.



## Stateful vs. Stateless Applications

### Stateful Applications
In a stateful architecture, the server is responsible for tracking the state of a client's interaction. This usually involves storing data about the current session in a database or cache (like Redis) and associating it with a unique identifier sent to the client.
*   **Example:** A shopping cart on an e-commerce site. When you add an item, the server stores that state in its memory or a database. As you navigate between pages, the server uses your session ID to retrieve your specific cart, ensuring your items remain saved.

*   **The Scaling Problem:** Every incoming request requires a lookup to retrieve the state, which can introduce latency. In a distributed system, maintaining a synchronized state across multiple servers or regions becomes a complex infrastructure challenge, often requiring "sticky sessions" or a shared data store.

### Stateless Applications
Statelessness means the server does not store any information about the client's previous requests. Each request must contain all the information necessary for the server to process it.

*   **Why we use it:** It enables effortless horizontal scaling. Because the server doesn't need to remember anything between requests, any instance of the application can handle any request. This decoupling is essential for microservices and high-concurrency environments where performance and reliability are critical.


---

### How do we solve this?
To achieve statelessness, the client must take responsibility for holding the necessary context and presenting it with every single request. In modern web applications, the most consistent way to handle this is by using a **Token**.

```json
{
  "user_info": {
    "user_id": 42,
    "user_name": "sam",
    "current_page" : "dashboard",
    "items_in_cart" : 2,
    "items" : [ 
      {"id": 1, "name": "paper"},
      {"id": 2, "name": "pen"}
       ],
    "logged_in" : "2026-02-12T10:08:33+05:30"
  }
}
```


Instead of the server maintaining a list of active sessions or interactions, it issues a token to the client. The client stores this token (usually in `localStorage` or an `HttpOnly` cookie) and attaches it to every subsequent request. This allows the server to process the request immediately using only the information provided in the token, without needing to query a central database for the user's current state.

An alarm bell might go off if your mind is already whirring with questions like when i apply this to a authz scenario, then :

> "What if the token is modified by the client themselves" 

or 
> "What if the token is stolen?" 

or
> "How do we invalidate a token?".

The answer to above problems is to use a special type of token called a **JWT** (JSON Web Token).

## JWT
A "Token" is a broad concept. It could be a random string of characters, an encrypted blob, or a json body but JWT is a special type of token that is compact, URL-safe, and can be used to represent claims to be transferred between two parties.


> **JWT has a lot of benefits, but the main power of JWT is that it cannot be tampered!**

---

## Creating a Token & Play around with it

Go to the website [jwt.io](https://jwt.io)

Immediately, you will be thrown into the "decoder" page - this is to debug an existing token ( which we will do so later) but we are here to create a token, so let's click on the "encoder" tab.

You will see 3 sections here :

1. **Header**: Algorithm & Token Type. ( Do not tamper this for now)
2. **Payload**: The data (Claims) - you can fill this up with any json body you wish.
3. **Sign JWT: Secret**: You are supposed to fill this up with a secret key. 

Once you fill up the above sections, you will see a token generated.

This has 3 parts separated by dots (`.`):

1. **Header**: Algorithm & Token Type.
2. **Payload**: The data (Claims).
3. **Signature**: Verifies the token hasn't been tampered with.

Essentially,
$$
jwt\_token = base64(header) + "." + base64(payload) + "." + signature
$$
where,
$$
signature = hash(base64(header) + "." + base64(payload), secret)
$$

This is the most simple JWT token that you can create.

You create the token, which has the "state" of the client. The client stores this token and sends it with every request. If the client modifies it you will get to know immediately!

> You can have your friend create a JWT token and you can place it in the **decoder** section now. Although you can read what he had kept in the body, you cannot modify it. If you return the same token to your friend, he will know if you have modified it. 

**This strongly implies, you should NEVER store any sensitive information in the token.**

---

### Side Quest 1: What does "Stateless AuthZ" actually mean?

To understand "stateless," we must first look at the "stateful" alternative: **Sessions**.

**1. The Stateful Way (Sessions)**
Imagine a club with a guest list.
- When you enter, the bouncer checks your ID and writes your name on a list inside the club (The Database/Redis).
- He gives you a simple ticket number `#123` (Session ID).
- Every time you want a drink, show ticket `#123`.
- The bartender must walk to the entrance, check the list for `#123`, see that it belongs to "Samridh", and then serve you.
- **Problem**: If the club gets huge and you add 10 bartenders (10 servers), they all need to check that ONE list. The list becomes a bottleneck.

**2. The Stateless Way (JWT)**
Imagine a club with stamped wristbands.
- When you enter, the bouncer checks your ID.
- He stamps a **detailed wristband** (The JWT) that says: *"Name: Samridh, Status: VIP, Valid until: 4 AM"*.
- He then **signs** it with an invisible ink that only staff can verify (The Digital Signature).
- Now, when you want a drink, you just show the wristband.
- The bartender checks the signature (is it real?) and reads the details directly from your wrist.
- **Benefit**: The bartender *never* needs to check a central list. You can have 1,000 bartenders across 5 floors, and none of them need to talk to each other.

> **Statelessness** means the server does not need to remember you. You bring your own credentials (state) with every single request.

---

## Side quest 2 

> You didn't allow me to tamper the first section of the token. Why?

The answer is : too keep things simple for now. Let's complicate our lives a bit.

So far, we have proven that JWT can be handed out to anyone and only YOU can tell apart if it's been modified or not. 

> But what if you want multiple people to verify that token hasn't been modified?

This is where you will change the first section, but default, it must show these values 

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

modify it to 

```json
{
  "alg": "RS256",
  "typ": "JWT"
}
```

The third section will automatically changes from `secret key` to `private key`.

Plugin a private key that you had learnt to create from [chapter 3](./03-asymmetric-cryptography.md). 

You will see that a new token is generated, if you move this to the decoder section now, then you will need a public key to verify it now.

So, this is what you do :

1. Generate the JWT token using your private key.
2. Send it to your client.
3. Put your public key in a well known place e.g., `https://your.company.com/.well-known/jwks.json`.
4. Now your friend who runs a different company can be sure that the token was generated by you and not by someone else.

These is extremely handy in microservices architecture as well as in multi-functional organization. 

e.g.,

you can login into the core banking website and then you can switch to it's insurance section with the same token, you will not require to login over and over again. 😁

Congratulations, you just learnt **Asymmetric JWT**!

---

## Alright, enough distractions, let's get back to the main topic : Roles and Permissions (RBAC)


> *RBAC stands for Role-Based Access Control.*

So, once the client is authorized, we simply add their `role` in the JWT payload and... they can't tamper it without us ( no priviledge escalation! ):

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "role": "admin",
  "iat": 1516239022
}
```

This is how we can "statelessly" enforce roles in our application.

These `roles` map to `permissions` in the backend, which allows the user to perform certain action in their platforms.

---
### Side Quest 3:  Critical Thinking: Why bother with Roles?

One might ask: *Can't we just list every specific permission (e.g., `create_user`, `edit_post`, `delete_report`) directly in the JWT for each user?*

While technically possible, roles act as a crucial layer of abstraction for several reasons:

1.  **Maintainability:** If your application has 50 different permissions and 10,000 users, managing individual mappings is a nightmare. With roles, you update the permissions for the `Editor` role once, and it immediately applies to all 500 editors.
2.  **JWT Payload Size:** JWTs are sent in every HTTP header. Listing 100 individual permissions would significantly increase the request size, leading to higher latency and potential header-size limit issues.
3.  **Business Logic Alignment:** Roles usually map to real-world job functions (e.g., "Accountant", "Moderator"). This makes the system easier for non-technical stakeholders to understand and audit.

In complex systems, you might see a hybrid approach: **RBAC** for broad categorization and **ABAC** (Attribute-Based Access Control) for fine-grained, context-aware permissions (e.g., "User can edit *this specific* post because they are the owner").

---

## Verifying tokens in code the modern way

In modern frameworks (Java Spring Boot, NestJS, .NET), we can use declarative annotations to enforce these roles, keeping our business logic clean.

**Example (Conceptual):**

```typescript
// Only 'Admin' role can delete users
@Roles('ADMIN')
@Delete('/users/:id')
deleteUser(id: string) {
  return db.delete(id);
}

// Any authenticated user can view profile
@Authenticated()
@Get('/profile')
getProfile(user: User) {
  return user.profile;
}
```

This works by using an **Interceptor** or **Middleware** that:
1. Intercepts the request.
2. Decodes and verifies the JWT.
3. Reads the `@Roles` metadata.
4. Checks if the `token.role` matches the required role.
5. Either allows the request or throws `403 Forbidden`.

---

> **But what if the token is stolen?**


If a token is stolen, the thief can impersonate the user until the token expires. Since the server is stateless, it doesn't "know" the token was stolen—it only knows the signature is valid.

To minimize risk, we use:
- **Short Expiration (`exp`):** Keep access tokens alive for only a few minutes (e.g., 15 mins).
- **Refresh Tokens:** A long-lived token used to issue new access tokens. These are typically stored in a database, allowing you to "revoke" a session by deleting the refresh token.
- **HTTPS:** Ensures the token isn't intercepted during transmission via Man-in-the-Middle attacks.

[**← Previous: PKIs & HTTPS**](./05-pkis-and-https.md) | [**🏠 Home**](../README.md)
