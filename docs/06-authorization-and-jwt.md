# Authorization & JSON Web Tokens (JWT)

> **Authentication (AuthN)** is establishing *who* you are.
> **Authorization (AuthZ)** is establishing *what* you can do.

Up until now, we've battled through encryption, trust, and handshakes (Authentication). You've proven *who* you are.

But just because you have an ID card, does it mean you can walk into the CEO's office? Or the server room? Or the vault?

That is the job of **Authorization**. It decides *where you can go* and *what you can do*.

But before we hand out the keys to the castle, we need to understand a fundamental architectural choice: **Stateful vs. Stateless**.



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

An alarm bell might go off if your mind is already whirring with questions like, "When I apply this to an AuthZ scenario, then...":

> *"What if the token is modified by the client themselves?"*

or 

> *"What if the token is stolen?"*

or

> *"How do we invalidate a token?"*

The answer to the above problems is to use a special type of token called a **JWT** (JSON Web Token).

## JWT
A "Token" is a broad concept. It could be a random string of characters, an encrypted blob, or a JSON body. A **JWT** is a special type of token that is compact, URL-safe, and can be used to transfer information between two or more parties without being tampered.


Basically, it's a JSON object that is digitally signed.

> **Think of the signature as a wax seal on a royal decree.**
> Any peasant can read the decree, but if they try to change a single letter, the seal breaks. The server is the only one with the signet ring to create a valid seal.

---

## Creating a Token & Playing around with it

Go to the website [jwt.io](https://jwt.io).

Immediately, you will be thrown into the "decoder" page—this is to debug an existing token (which we will do later). But we are here to create a token, so let's look at the interface.

You will see 3 sections here:

1. **Header**: Algorithm & Token Type (Do not tamper with this for now).
2. **Payload**: The data (Claims) - you can fill this in with any JSON body you wish.
3. **Verify Signature**: You are supposed to fill this in with a secret key. 

Once you fill in the above sections, you will see a token generated.

This new "token" has 3 parts separated by dots (`.`):

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

You create the token, which encapsulates the "state" of the client. The client stores this token and sends it with every request. If the client modifies it, you will know immediately!

> You can have a friend create a JWT and place it in the **decoder** section. Although you can read what they kept in the body, you cannot modify it. If you return the modified token to your friend, they will know if you have tampered with it.

**This strongly implies: you should NEVER store any sensitive information in the token.**

---

### Side Quest 1: What does "Stateless AuthZ" actually mean?

To understand "stateless," we must first look at the "stateful" alternative: **Sessions**.

**1. The Stateful Way (The Amnesiac Receptionist with a Ledger)**
Imagine a club where the bouncer has zero memory of faces.
- When you enter, he writes your name in a giant ledger: "Samridh is inside."
- He hands you a ticket `#123`.
- You want a drink? Show ticket `#123`.
- The bartender runs to the door, checks the ledger for `#123`, confirms it's Samridh, and pours the drink.
- **The Problem:** If the club gets huge and you have 10 bartenders, they all need to constantly check that *one* ledger. The line at the ledger becomes your bottleneck.

**2. The Stateless Way (The VIP Wristband)**
This time, the bouncer still has amnesia, but he's smarter.
- When you enter, he checks your ID.
- He doesn't write anything down. Instead, he stamps a **detailed wristband** (The JWT) that says: *"Name: Samridh, Status: VIP, Valid until: 4 AM"*.
- He puts a special **holographic seal** on it that only staff can verify (The Signature).
- You want a drink? Show the wristband.
- The bartender checks the seal (is it real?) and reads: "Ah, VIP Samridh. Here's your drink."
- **Benefit:** The bartender *never* needs to talk to the bouncer or check a central list. You can have 1,000 bartenders across 5 floors, and they can all work independently. This is infinite scalability.

> **Statelessness** means the server does not need to remember you. You bring your own credentials (state) with every single request.

---

## Side Quest 2: Asymmetric Keys

> You didn't allow me to tamper with the first section of the token. Why?

The answer: It was to keep things simple until now. Now let's complicate our lives a bit.

So far, we have proven that a JWT can be handed out to anyone and only **YOU** can tell if it's been modified.

> But what if you want *multiple people* to verify that the token hasn't been modified?

This is where you will change the first section. By default, it shows these values:

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

The third section will automatically change from `secret key` to `private key`.

Plug in a private key that you learned to create in [Chapter 3](./03-asymmetric-cryptography.md). 

You will see that a new token is generated. If you move this to the "decoder" section, you will need a public key to verify it.

So, this is what you do:

1. Generate the JWT using your private key.
2. Send it to your client.
3. Put your public key in a well known place e.g., `https://your.company.com/.well-known/jwks.json`.
4. Now your friend who runs a different company can be sure that the token was generated by you and not by someone else.

This is extremely handy in microservices architecture as well as in multi-functional organizations. 

For example:
You can log in to the core banking website and then switch to its insurance section with the same token; you will not be required to log in over and over again. 😁

Congratulations, you just learned **Asymmetric JWT**!

---

## Alright, enough distractions, let's get back to the main topic : Roles and Permissions (RBAC)


> *RBAC stands for Role-Based Access Control.*

So, once the client is authorized, we simply add their `role` in the JWT payload and... they can't tamper with it without us detecting it (no privilege escalation!):

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "role": "admin",
  "iat": 1516239022
}
```

This is how we can "statelessly" enforce roles in our application.

These `roles` map to specific permissions in the backend code, controlling exactly what buttons a user can click and what data they can see.

---
### Side Quest 3:  Critical Thinking: Why bother with Roles?

One might ask: *Can't we just list every specific permission (e.g., `create_user`, `edit_post`, `delete_report`) directly in the JWT for each user?*

While technically possible, roles act as a crucial layer of abstraction for several reasons:

1.  **Maintainability:** If your application has 50 different permissions and 10,000 users, managing individual mappings is a nightmare. With roles, you update the permissions for the `Editor` role once, and it immediately applies to all 500 editors.
2.  **JWT Payload Size:** JWTs are sent in every HTTP header. Listing 100 individual permissions would significantly increase the request size, leading to higher latency and potential header-size limit issues.
3.  **Business Logic Alignment:** Roles usually map to real-world job functions (e.g., "Accountant", "Moderator"). This makes the system easier for non-technical stakeholders to understand and audit.

In complex systems (like AWS IAM), you often see a hybrid approach: **RBAC** for broad categories (Admin, User) and **ABAC** (Attribute-Based Access Control) for fine-tuning (e.g., "User can edit *this specific document* because their ID matches the `owner_id` field").

---

## Verifying tokens in code the modern way

In modern frameworks (Java Spring Boot, NestJS, .NET), we can use declarative annotations to enforce these roles, keeping our business logic clean.

---

### Example

This is what you get from the client side:

**HTTP Header**
```http
POST /api/provision/newInstance HTTP/1.1
Host: my.company.com
Content-Type: application/json
Content-Length: 27
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6InNhbXJpZGgudHVsYWRoYXJAZ21haWwuY29tIiwicm9sZXMiOlsicm9sZXMvY29tcHV0ZS5hZG1pbiIsInJvbGVzL25ldHdvcmtpbmcudXNlciJdLCJpYXQiOjE1MTYyMzkwMjJ9.tzLCV-7e3y33m48BC5v6WTSKSH1RvgJTHZzWzp_baew
```

**HTTP POST body**
```
{
    "instanceName": "samridh-instance",
    "instanceType": "t2.micro",
    "instanceRegion": "us-east-1"
}
```
---

### Backend Logic

The most trivial way is to have a function that does the verification of the JWT token and return the permissions allowed for the role, then check if the user has the required permission to perform the action.

```java

public boolean isJWTTokenValid(String token) {
  return someFunctionThatVerifiesTheToken(token);
}

public Permission[] getPermissions(String token) {
  return someFunctionThatReturnsPermissions(token);
}

public boolean hasPermission(Permission[] permissions, String requiredPermission) {
  for (Permission p : permissions) {
    if (p.name.equals(requiredPermission)) {
      return true;
    }
  }
  return false;
}

  
@PostMapping("/provision/newInstance")
public String provisionNewInstance(@RequestHeader("Authorization") String authHeader, @RequestBody Instance instance) {
  String token = authHeader.replace("Bearer ", "");

  if (isJWTTokenValid(token)) {
    Permission[] permissions = getPermissions(token);
    if (hasPermission(permissions, "provision/newInstance")) {
      functionToProvisionInstance(instance); 
      return "Instance provisioned successfully";
    } else {
      return "User does not have permission to provision instance";
    }
  } else {
    return "Invalid JWT token";
  }
}


```

But this isn't cool, nor is it the best way to do it. This causes a lot of boilerplate code.

You should implement this using **Interceptors**, **Middleware**, **Aspect Oriented Programming (AOP)**, or **Decorators**.

> **Interceptors**, **Middleware**, **AOP**, and **Decorators** are conceptually similar patterns used to handle cross-cutting concerns (like checking a token). Pick your weapon of choice:

| Pattern Name | Common Ecosystem |
|:---|:---|
| **Interceptors** | Java Spring Boot, gRPC |
| **Middleware** | Node.js Express, Go Gin |
| **AOP** | Java (Spring AOP), .NET |
| **Decorators** | Python (Flask/Django), TypeScript (NestJS) |

Keep the previous functions that you have used. Simply upgrade them to annotations that can be used everywhere. Converting a function to an annotation isn't covered in this repository as it's not strictly a security topic, so you're on your own for that part!

this is the final code base that you should be having ( the first three functions remain intact ):

```java

@PostMapping("/provision/newInstance")
@Authorized(permissions = "provision.instances")
public String provisionNewInstance(@RequestBody Instance instance) {
  functionToProvisionInstance(instance); 
  return "Instance provisioned successfully";
 
}

```

The `@Authorized` annotation is a custom annotation that is used to enforce role-based access control. It is used to specify the permissions that are required to access a particular resource, and it should handle all the work of the three functions in one shot.

This is a very basic example of how to use the `@Authorized` annotation, and it can be used in many different ways.

Since this documentation was made only for security related documentations, I am skipping out on the details of creating a proper production grade RBAC system, and just sticking to theory.

Consider annotations as magical wrappers—they're powerful, but if you don't understand the underlying AOP proxy mechanism, you might accidentally bypass security! Always test your "unauthorized" paths as rigorously as your "authorized" ones.

---

## The Achilles' Heel of Statelessness: Stolen Tokens or Replayed Tokens

Here is the scary part.


If a token is stolen or replayed, the thief can impersonate the user until the token expires. Since the server is stateless, it doesn't "know" the token was stolen—it only knows the signature is valid.

To mitigate the risk of token theft and replay attacks without reintroducing server-side state, we utilize cryptographic and temporal constraints:

#### 1. Short-Lived Access Tokens (`exp`)
The primary stateless defense is minimizing the `exp` (expiration) window.
*   **Mechanism:** Access tokens are issued with a short lifespan (e.g., 5–10 minutes).
*   **Refresh Tokens:** To maintain user sessions, a long-lived Refresh Token is used to request new Access Tokens. This ensures that an intercepted Access Token has a very limited window of utility.

#### 2. Sender Constrained Tokens (DPoP)
**Demonstration of Proof-of-Possession (DPoP)** binds the JWT to the client's cryptographic identity, making the token non-transferable.
*   **Step 1:** The client generates an asymmetric key pair and provides the public key to the server during token issuance.
*   **Step 2:** The server embeds a thumbprint of this public key in the JWT (the `cnf` claim).
*   **Step 3:** For every request, the client generates a "DPoP proof"—a unique, short-lived JWT signed by their private key that covers the specific HTTP method and target URI.
*   **Step 4:** The resource server verifies that the signature on the

---

[**← Previous: PKIs & HTTPS**](./05-pkis-and-https.md) | [**🏠 Home**](../README.md)
