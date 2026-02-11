# Hashing & Digital Signatures

## Hashes

A **hash** is a mathematical function that converts an input (or 'message') of any length into a fixed-size string of bytes. The output (often called the `digest`) looks completely random, but it is **deterministic**—the same input will always produce the exact same hash.

With simple hashing techniques, it can be easy to figure out the input even if you only have the output. However, a **cryptographic hash function** is designed to be computationally infeasible to reverse (i.e., find the input given the output).

$$
hash = H(message)
$$

### Why were they needed?

1.  **Integrity**: How do you know a file you downloaded hasn't been corrupted or tampered with? Comparing the file byte-for-byte is slow. Comparing their hashes is instant.

> The source sends the main file as well as its hash string. The destination re-computes the hash of the received file and compares it with the hash string sent by the source.

2.  **Efficiency in Signing**: As mentioned in the RSA section above, encrypting a massive file (like a novel or video) with a private key is computationally expensive. Instead, we hash the file (which is fast) and sign the hash (which is small).
3.  **Password Security**: Storing passwords in plain text is dangerous. If we store the hash of the password, even if the database is leaked, the attacker only gets the hashes, not the actual passwords. (Since we use cryptographic hashes, it is computationally infeasible to reverse the hash to retrieve the password).

### Usage

- **Digital Signatures**: Combining Hashes + Asymmetric Encryption (RSA) = Digital Signature.
- **Commit IDs**: Git uses SHA-1 hashes to uniquely identify the state of the code.
- **File Integrity**: Verifying checksums of downloaded files.

### Generating a Hash using OpenSSL

We can use the `openssl dgst` (digest) command. Here is an example using `SHA-256`:

```shell
# Note: echo adds a newline by default, use -n to suppress it for exact string hashing
echo -n "hello world" | openssl dgst -sha256
# Output: (stdin)= b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9
```

> **Note**: `openssl` uses a cryptographic hash function, so if you try changing just one letter, you will see that the hash changes completely. This is called the **Avalanche Effect**, making it extremely difficult to find patterns or trace the source.

```shell
echo -n "Hello world" | openssl dgst -sha256
# Output: (stdin)= 64ec88ca00b268e5ba1a35678a1b5316d212f4f366b2477232534a8aeca37f3c
```

> There is nothing wrong with "classical" hash functions—they have their own uses, especially in memory management (like hash maps). But when dealing with security, we almost always use **cryptographic** hash functions. **Side quest completed!**

---

[**← Previous: RSA**](./03-rsa-and-asymmetric.md) | [**🏠 Home**](../README.md) | [**Next: PKIs & HTTPS →**](./05-pkis-and-https.md)
