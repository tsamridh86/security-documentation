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

## What is a JWT?

JWT (JSON Web Token) is a compact, URL-safe means of representing claims to be transferred between two parties.

It consists of three parts separated by dots (`.`):
1. **Header**: Algorithm & Token Type.
2. **Payload**: The data (Claims).
3. **Signature**: Verifies the token hasn't been tampered with.

$$
token = base64(header) + "." + base64(payload) + "." + signature
$$

---

## Creating a Token

### 1. Symmetric Signature (HMAC / HS256)
This uses a single secret key. The server signs the token with a secret, and later verifies it with the same secret. Fast and simple, but the secret must be kept safe.

```javascript
/* Pseudocode */
// SIGNING
signature = HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  "my_secret_key"
)
```

### 2. Asymmetric Signature (RSA / RS256)
This uses a Key Pair (Private/Public), just like we learned in [Chapter 3](./03-rsa-and-asymmetric.md).
- **Sign** with the **Private Key** (Auth Service).
- **Verify** with the **Public Key** (Any Microservice).

This is powerful for distributed systems. The Auth server holds the private key and issues tokens. Other services only need the public key to verify the token is valid; they cannot forge new tokens.

---

## Why is JWT required?

1. **Statelessness**: The server doesn't need to store a session ID in a database (like Redis). The token itself contains all the user info (`user_id`, `role`, `expiry`).
2. **Scalability**: Since there is no database lookup to check if a user is logged in, you can scale your services horizontally easily.
3. **Cross-Domain / Microservices**: Pass the token between services effortlessly.

### Side Quest: What does "Stateless" actually mean?

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

## Roles and Permissions (RBAC)

We usually include a `role` or `permissions` claim in the JWT payload:

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "role": "admin",
  "iat": 1516239022
}
```

### Implementing with Annotations

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

[**← Previous: PKIs & HTTPS**](./05-pkis-and-https.md) | [**🏠 Home**](../README.md)
