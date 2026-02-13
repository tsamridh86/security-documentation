# JWT Implementation & Security

> In the [previous chapter](./06-authorization-concepts.md), we learned about the concepts of Authorization, Statelessness, and the basic anatomy of a JWT. Now, let's dive into advanced implementation details, asymmetric signing, and security best practices.

---

## Alright, enough theory, let's return to the main topic : Roles and Permissions (RBAC)


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
### Side Quest 1:  Critical Thinking: Why bother with Roles?

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

## The Kryptonite of Statelessness: Stolen Tokens or Replayed Tokens

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


## Side Quest 2: "What's that Log in with Google?" (OAuth 2.0)

> *Wait, if I use 'Log in with Google', does that mean I'm giving my Google password to this random app?*

**NO!** And that is the brilliance of **OAuth 2.0**.

OAuth 2.0 is a protocol for **Delegation**.

Imagine you are staying at a fancy hotel (Google).
You want to order pizza from a local shop (The App) to your room.
You do *not* give the pizza guy your master room key card (Password).
Instead, you go to the front desk and get a **Valet Key** (Access Token).
You give this Valet Key to the pizza guy. It allows him to access *only* the elevator and *only* your floor for 10 minutes.

**In tech terms:**
1.  **Resource Owner (You):** You want to use an App.
2.  **Client (The App):** "Hey Google, can I access Sam's email?"
3.  **Authorization Server (Google):** "Sam, do you trust this App to see your email?"
4.  **You:** "Yes."
5.  **Google:** "Okay App, here is an **Access Token**. It is valid for 1 hour and can *only* read emails."

The App never sees your password. It only gets a token with limited permissions.

> **OpenID Connect (OIDC)** is a thin layer on top of OAuth 2.0 that handles **Authentication** (checking who you are), while plain OAuth 2.0 handles **Authorization** (checking what the app is allowed to do).

More readup about the actual protocol here : https://auth0.com/docs/get-started/authentication-and-authorization-flow

---

[**← Previous: Authorization Concepts**](./06-authorization-concepts.md) | [**🏠 Home**](../README.md) | [**Next: Other Forms of Attacks →**](./08-other-forms-of-attacks.md)
