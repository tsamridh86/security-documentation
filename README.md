# The definitive guide to digital security

Everything I know about digital security in one place!

Digital security for engineering often seems daunting and difficult to deal with. That is why I've written this article: to make it accessible to everybody and keep it extremely practical, relating all the math, numbers, and certificates to real life.

---

## 📖 The Story

For those who prefer a narrative approach, I have re-written the concepts below into a thriller story set in a cyberpunk Pune.

👉 [**Read "The Rogue's Protocol" (Story Mode)**](./story.md)

---

## 📚 Technical Guide (The Index)

I have split the guide into focused chapters to make it easier to digest. You can read them in order or jump to specific topics.

### [1. History & Symmetric Encryption](./docs/01-history-and-basics.md)
   - The Spartan "Scytale" Cipher
   - Basics of Encryption vs Encoding
   - Modern Symmetric Encryption (AES)

### [2. The Key Exchange Problem (Diffie-Hellman)](./docs/02-diffie-hellman.md)
   - The weakness of a single key
   - The "Paint Mixing" Trick (Modular Arithmetic)
   - Diffie-Hellman Key Exchange Demo

### [3. Asymmetric Encryption (RSA)](./docs/03-rsa-and-asymmetric.md)
   - Man-in-the-Middle Attacks
   - RSA: Public and Private Keys
   - Confidentiality vs Authenticity

### [4. Hashing & Digital Signatures](./docs/04-hashing-and-signatures.md)
   - What is a Hash?
   - Integrity and Password Security
   - Signing messages with RSA

### [5. PKIs, Certificates & HTTPS](./docs/05-pkis-and-https.md)
   - The "Trust" Problem
   - Certificate Authorities (CAs) and CSRs
   - The full HTTPS Handshake workflow

### [6. Authorization & JWTs](./docs/06-authorization-and-jwt.md) - still under construction! not added to the story yet!
   - Symmetric vs Asymmetric Signatures
   - Why JWT? (Statelessness)
   - Roles, Permissions & Annotation-based Security

---