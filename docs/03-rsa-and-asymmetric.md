# RSA & Asymmetric Encryption

## Another problem with the key exchange!

### How do I trust that the key wasn't intercepted in the middle?

After all, if I perform this key exchange with someone impersonating the destination, then I'm exposed to a **"man-in-the-middle attack"**.

> **Solution: RSA Algorithm**

---

## RSA Algorithm

A full read-up is available here: https://simple.wikipedia.org/wiki/RSA_algorithm

Internal mathematics have been skipped for now.

Mathematically, this is what the RSA algorithm does:

$$
cipher = rsa(msg, private\_key)
$$

$$
msg = rsa(cipher, public\_key)
$$

> **Note**: A message encrypted by a private key can only be opened by the publicly available key. This establishes **Authenticity** -> Only one person on the planet could have written that message.

Conversely,

$$
cipher = rsa(msg, public\_key)
$$

$$
msg = rsa(cipher, private\_key)
$$

> **Note**: A message encrypted by the public key can only be opened by the private key. This establishes **Confidentiality** -> Only one person in the world will ever read that message.

---

### Practical demo

1. Generate a private key & public key:
```shell
openssl genpkey -algorithm RSA -out private.pem -pkeyopt rsa_keygen_bits:2048

openssl rsa -pubout -in private.pem -out public.pem
```
2. Encrypt a message with the public key:
```shell
echo "Hello, RSA!" | openssl rsautl -encrypt -pubin -inkey public.pem -out encrypted.bin
```
3. Have a look at the output:
```shell
openssl base64 -in encrypted.bin
```
4. Decrypt the message with the private key:
```shell
openssl rsautl -decrypt -inkey private.pem -in encrypted.bin
```
---

> **Critical Thinking**: If I encrypt a message using the private key, and anyone can decrypt it using the publicly available key... why should I waste compute power encrypting large input messages?

> **Answer**: Think at the scale of proving authorship used in signing. If you want to prove that you wrote a novel, you don't need to sign the entire text. Instead, you only sign the **hash** of the input. That's plenty to prove **authenticity**.

Thankfully, the creators of `openssl` had the same smart thinking. In `openssl`, you can only **sign** a message using the private key; there is no facility to "encrypt" a message using the private key (which is effectively signing).

This implies:

$$
    digitalSignature = rsa(hash(message), private\_key)
$$

> The command to generate a signature is left as an exercise.

---

[**← Previous: Diffie-Hellman**](./02-diffie-hellman.md) | [**🏠 Home**](../README.md) | [**Next: Hashing & Signatures →**](./04-hashing-and-signatures.md)
