# security-documentation
Best of my knowledge about digital security in one place!


Digital security for engineering seems daunting and a difficult subject to deal with : this is why I've written this article - to make it accessible to everybody and keep it extremely practical, to actually relate to all the math and the numbers and certificates with real life.

## History

So, let's begin, let's start the story about 3000 years ago:
The first instance of encrypted was noted around the era of "Alexander the Great" - because he had a problem : his territories were so large that he couldn't be everywhere at once; he needed to send messages to his trusted generals, so that they are able to execute this commands. 

However, if this message is intercepted in the middle, then the strategy fails. 

Hence, he came up with this rather simple solution :

![Scytale cipher](https://upload.wikimedia.org/wikipedia/commons/5/51/Skytale.png)

A rope is wrapped around on a stick of a known diameter, then a message is written out on only one line, say "Attack at dawn" and the others are completely gibberish.
Then the rope is unwrapped and sent to the generals, unless you have the stick with the same diameter as the original, there is no chance that you are going to read the message ( well, at least 3000 years ago when there were no computers and education was not common ).


But let us observe what happened there,

Mathematically,

$$
cipherText = f ( msg , key )
$$

$$
msg = f' ( cipherText , key )
$$

The above technique, had a "function" that would convert the readable message to an unreadable message for a normal person, however, if you have a key, then this "cipher text" can be recovered to the original message.


This method of converting a message to an unreadable format unless you have a key is known as **"encryption"** & if you have a singular key that does the encryption & decryption it's known as **"symmetric encryption"**

---

Remember, these key differences :


- **encryption** : is the activity to mutate ( encrypt ) your data such that even though if it's left in public, then only the person with the key should be able to read ( decrypt ) it.
- **encoding** : is just a translation from one characters to another - this is typically done to avoid special characters, esp. in scenarios where new characters are unsupported. e.g., using emoji's in legacy database "😂" emoji is encoded to : U+1F602 to store in legacy database and the frontend will take the headache of converting it back to emoji 😂.
- **steganography** : is the activity where your data is hidden amongst other larger set of data.

this article will stick to encryption - no usage of encoding or steganography here!

---

## Example of modern symmetric encryptions 

Of course the scytale cipher isn't complex to break anymore, but we do use better symmetric encryption algorithms now. The most popular one is AES ( Advanced Encryption Standard ).

### Encryption :

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

- tampering the key or the cipher text will always lead to an error.
- in modern encrpytion algorithms, attempting the same encrpytion repeatedly. i.e., using the same key & message may still lead to different output - this trick is known as `salting` - this will not be covered in this article, perhaps another `.md` file.
- modern encryption algorithms, "chain" blocks due to advanced pattern recoginition software that is available now, this trick is known as `cipher-block-chaining` , or `CBC` e.g.,


  ![CBC example](https://miro.medium.com/1*WzF5Rcsnb8gn3JmM-uumTQ.png)

If poorly encrypted, then even the best algorithm can't help!

---

## Main weakness of symmetric encryption

> that KEY - it needs to be transferred securely, if the key is leaked then the entire exchange is compromised.

### how then, shall we securely exchange a key when the messenger is unreliable?

> by using a 8th standard mathematical trick.


$$
((u)^x)^y = ((u)^y)^x
$$
$$
u^{xy} = u^{yx}
$$


