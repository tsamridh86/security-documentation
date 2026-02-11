# History and Symmetric Encryption

So, let's begin the story about 3000 years ago.

The first instance of **encryption** was noted around the era of the Spartans (approx 400 BC). They had a problem: their territories were effectively managed by generals who needed to receive secret commands.

However, if a message were intercepted in the middle, the strategy would fail.

Hence, he came up with a rather simple solution:

![Scytale cipher](https://upload.wikimedia.org/wikipedia/commons/5/51/Skytale.png)

A rope is wrapped around a stick of a known diameter. A message is then written on only one line, say "Attack at dawn," while the other lines contain gibberish.
When the rope is unwrapped, it appears as random letters. Unless you have a stick with the same diameter as the original (the "key"), there is no chance you are going to read the message (well, at least 3000 years ago when computers didn't exist).


Let us observe what happened there.

Mathematically,

$$
cipherText = f ( msg , key )
$$

$$
msg = f' ( cipherText , key )
$$

The above technique used a "function" that would convert the readable message into an unreadable format for a normal person. However, if you have the key, then this "ciphertext" can be recovered to the original message.


This method of converting a message to an unreadable format is known as **"encryption"**. If you have a singular key that does both the encryption and decryption, it's known as **"symmetric encryption"**.

---

Remember these key differences:


- **Encryption**: The activity of mutating (encrypting) your data so that even if it's public, only the person with the key can read (decrypt) it.
- **Encoding**: A translation from one set of characters to another. This is typically done to avoid special characters, especially where new characters are unsupported. For example, using emojis in a legacy database: the "😂" emoji might be encoded to `U+1F602` to store it, and the frontend handles converting it back to "😂".
- **Steganography**: The activity of hiding data within another larger set of data.

This article will stick to **encryption**—no encoding or steganography here!

---

## Example of modern symmetric encryption

Of course, the scytale cipher isn't complex to break anymore, but we use better symmetric encryption algorithms now. The most popular one is **AES** (Advanced Encryption Standard).

### Encryption:

```shell
echo "your message" | openssl enc -aes-256-cbc -salt -pbkdf2 -a -pass pass:yourkey

# output : U2FsdGVkX1/l5llR2vDIoFCz1Ysbk99hGSwZAEOcjGs
```

### Decryption

```shell
echo "U2FsdGVkX1/l5llR2vDIoFCz1Ysbk99hGSwZAEOcjGs" | openssl enc -d -aes-256-cbc -pbkdf2 -a -pass pass:yourkey

# Output : your message
```

### Points to note and experiment

- Tampering with the key or the ciphertext will always lead to an error.
- In modern encryption algorithms, attempting the same encryption repeatedly (i.e., using the same key & message) may still lead to different output. This is due to `salting`, which prevents attackers from seeing patterns.
- Modern encryption algorithms "chain" blocks together to prevent pattern analysis. This is known as `Cipher-Block-Chaining` or `CBC`. For example:


  ![CBC example](https://miro.medium.com/1*WzF5Rcsnb8gn3JmM-uumTQ.png)

If poorly encrypted, then even the best algorithm can't help!

---

[**🏠 Home**](../README.md) | [**Next: Diffie-Hellman Key Exchange →**](./02-diffie-hellman.md)
