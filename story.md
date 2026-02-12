# The Rogue's Protocol: A Tale of Shadows and Silicon

## Prologue: The Faraday Cafe

The neon smog of **Neo-Pune** choked the skyline. Anaya and Vikram sat in the back of a "Faraday Cafe" in the rusted underbelly of Shivajinagar. The walls were lined with copper mesh; it was the only place they could speak face-to-face without **The Syndicate** listening.

"The Logic Bomb is planted," Anaya whispered, tapping the dormant server blade on the table before sliding it into her backpack. "I'm installing this inside the Syndicate's Hinjewadi Mainframe tonight. Once it's online, I need you to trigger it from the outside."

Vikram, a disgraced cyber-samurai with a carbon-fiber arm, frowned. "Triggering it requires sending the payload across the net. The Syndicate monitors *every* packet. If they see the command, they trace it back to us."

"That's why we need a protocol," Anaya said. "A way to send the command so they see nothing but noise."

---

## Chapter 1: The Lesson of the Stick (History)

Vikram pulled up a holographic history file. "We aren't the first to face this. 3000 years ago, Spartan generals used a **Scytale**."

![Scytale cipher](https://upload.wikimedia.org/wikipedia/commons/5/51/Skytale.png)

"They wrapped a strip of parchment around a stick of a known diameter. The message 'Attack at dawn' only appeared when wrapped around the correct stick. To anyone else, it was gibberish."

Mathematically, it was the birth of encryption:

$$
cipherText = f ( msg , key )
$$

$$
msg = f' ( cipherText , key )
$$

"But the Syndicate doesn't use sticks," Anaya noted. "They use supercomputers."

---

## Chapter 2: The Failed Courier (Symmetric Encryption)

*Two days later.* 

Anaya had successfully planted the node (The Server) inside the mainframe's cooling vents. Vikram was at a safehouse in Kothrud, ready to send the activation codes.

They had agreed on a plan: **Symmetric Encryption** using **AES** (Advanced Encryption Standard).

"I'll encrypt the payload," Vikram thought. "Even if they intercept it, they can't read it without the key."

He typed the command:

```shell
echo "EXECUTE_Protocol_Zero" | openssl enc -aes-256-cbc -salt -pbkdf2 -a -pass pass:ChaiNiMaska
```

The output was scrambled:
```
U2FsdGVkX1/l5llR2vDIoFCz1Ysbk99hGSwZAEOcjGs
```

To decrypt it, the receiver would need the exact command:
```shell
echo "U2FsdGVkX1/l5llR2vDIoFCz1Ysbk99hGSwZAEOcjGs" | openssl enc -d -aes-256-cbc -pbkdf2 -a -pass pass:ChaiNiMaska
```

Vikram felt secure because of **Cipher-Block-Chaining (CBC)**. It chained blocks together so that any tampering would break the chain.

![CBC example](https://miro.medium.com/1*WzF5Rcsnb8gn3JmM-uumTQ.png)

**The Problem:** The Server (Anaya's Node) needed the password `ChaiNiMaska` to decrypt it.

Vikram couldn't send the password over the net. So, he gave it to **Rohan**, a physical runner. "Get within range of the Hinjewadi tower," Vikram ordered. "Upload the key via short-range burst."

Rohan never made it. Syndicate drones intercepted him at the perimeter. They extracted the key from his neurolink before stopping his heart. The Syndicate used the key to decrypt the test packets Vikram had sent.

They had failed. **Lesson 1: Symmetric Encryption is useless if you can't share the key securely.** (And couriers are mortal).

---

## Chapter 3: The Ghost in the Wire (Diffie-Hellman)

*Back at the Faraday Cafe.*

Anaya looked tired. "We lost Rohan. We can't risk people anymore."

"Then we use math," Vikram slammed his metal fist on the table. "We use the **Diffie-Hellman Key Exchange**. We can generate a shared key over the public network. Takes the courier out of the equation."

Vikram explained the logic. It relied on **Modular Arithmetic**.

$$
((g)^x)^y = ((g)^y)^x
$$
$$
g^{xy} = g^{yx}
$$

*That night, the attempt began.*

```mermaid
sequenceDiagram
    participant V as Vikram (Client)
    participant A as Node (Server)
    
    Note over V,A: Setup phase (public parameters known)
    
    V->>V: Generates secret x
    A->>A: Generates secret y
    
    V->>A: Sends g^x (Syndicate sees this)
    A->>V: Sends g^y (Syndicate sees this)
    
    V->>V: Computes Key
    A->>A: Computes Key
```

Vikram ran the simulation with simple numbers to be sure:
*   $g = 5$ (Base)
*   $p = 135$ (Modulo)
*   $x = 8$ (Vikram's Secret)
*   $y = 9$ (Node's Secret)

![DFHKE](https://github.com/tsamridh86/security-documentation/blob/main/dhke.gif?raw=true)

1. Vikram sent $g^x \pmod p$ -> `70`
2. Node sent $g^y \pmod p$ -> `80`
3. Vikram calculated $(80)^8 \pmod {135} = 55$
4. Node calculated $(70)^9 \pmod {135} = 55$

They had a shared key: `55`.

**The Failure:**
The Syndicate's AI, **The Overseer**, was faster. It intercepted Vikram's `70`. It pretended to be the Node and sent its own number back. It then pretended to be Vikram and spoke to the Node.

The Overseer sat in the middle, decrypting Vikram's messages, reading "EXECUTE_Protocol_Zero", and blocking them.

Vikram stared at his screen. "Connection Timeout."

**Lesson 2: You can secure the channel, but without identity, you might be securing a channel with the enemy.** (Man-in-the-Middle).

---

## Chapter 4: The Digital Fingerprint (RSA)

*Faraday Cafe. Ten hours to the deadline.*

"We need to know who we're talking to," Anaya said, rubbing her temples. "The Node needs to prove it's *my* Node, not a Syndicate honeypot. And you need to be sure only *my* Node can read the command."

"**RSA**," Vikram said. "Public and Private keys."

"To establish **Confidentiality** (Secrecy):"
$$
cipher = rsa(msg, public\_key)
$$
$$
msg = rsa(cipher, private\_key)
$$

"To establish **Authenticity** (Identity):"
$$
cipher = rsa(msg, private\_key)
$$
$$
msg = rsa(cipher, public\_key)
$$

"I'll encrypt the command with the Node's **Public Key** so only the Node can read it. And the Node will sign its responses with its **Private Key** so I know it's real."

Anaya remotely updated the Node's firmware to generate keys:

```shell
openssl genpkey -algorithm RSA -out node_private.pem -pkeyopt rsa_keygen_bits:2048
openssl rsa -pubout -in node_private.pem -out node_public.pem
```

But Vikram hesitated. "How do I get the Node's Public Key? If the Node sends it to me, the Overseer could intercept it and send *his* fake Public Key instead."

They were back to the trust problem.

---

## Chapter 5: The Silent Swami (Certificate Authority)

"We need a higher power," Anaya said. "A **Certificate Authority**."

They contacted **The Rishi**, a legendary hacker collective that acted as the root of trust for the resistance. The Rishi's "Root Certificate" was already hardwired into Vikram's deck (`/etc/ssl/certs`).

**Step 1:** Anaya ordered the Node to generate a **Certificate Signing Request (CSR)**.

```shell
openssl req -new -sha256 -nodes -out node.syndicate.net.csr -newkey rsa:2048 -keyout node_private.key -config <(
cat <<-EOF
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
req_extensions = req_ext
distinguished_name = dn

[ dn ]
C = IN
ST = Maharashtra
L = Neo-Pune
O = The Resistance
OU = Ops
CN = node.syndicate.net

[ req_ext ]
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = node.syndicate.net
EOF
)
```

**Step 2:** Anaya routed this request to The Rishi via a dead-drop.

**Step 3:** The Rishi verified Anaya's signature and signed the Node's certificate using their Root Key.

*(The Rishi's Setup - done previously)*:
```shell
openssl req -x509 -sha256 -nodes -days 3650 -newkey rsa:4096 -keyout rishi.key -out rishi.crt
```

*(The Signing)*:
```shell
openssl x509 -req -in node.syndicate.net.csr -CA rishi.crt -CAkey rishi.key -CAcreateserial -out signed_certificate.crt -days 365 -sha256
```

Now, the Node had a Digital ID Card (`signed_certificate.crt`) signed by The Rishi.

**The Final Setup (HTTPS):**

Anaya programmed the Node (The Server) to present this certificate to anyone who connected.

```javascript
// The Logic Bomb Node (Server)
const https = require('https');
const fs = require('fs');

const options = {
  key: fs.readFileSync('node_private.key'),
  cert: fs.readFileSync('signed_certificate.crt') // Signed by The Rishi
};

https.createServer(options, (req, res) => {
  // If we get here, the tunnel is secure.
  res.writeHead(200);
  res.end('System Ready. Awaiting Command.');
}).listen(443);
```

---

## Chapter 6: The Heist

Vikram sat on a rooftop in Magarpatta, rain slicking his deck. The Syndicate's drones circled overhead. This was it.

He initiated the connection to the Node inside the Mainframe.

**Step 1: Handshake**
Vikram's Deck: "Hello?"
Node: "Here is my Certificate."
Vikram's Deck: *Checks signature against The Rishi's Root.* **VALID.**

![Checking certificate](https://github.com/tsamridh86/security-documentation/blob/main/certificate-check.gif?raw=true)

**Step 2: The Tunnel**
They performed the Diffie-Hellman exchange *inside* this authenticated session. The Overseer tried to interfere, but couldn't fake the certificate.

The Secure Tunnel was established.

```mermaid
sequenceDiagram
    participant Node
    participant CA as Rishi
    participant Vikram

    Note over Vikram: Browsing to https://syndicate-mainframe.node
    Node->>Vikram: Sends Signed Certificate
    Vikram->>Vikram: Verifies Signature (Trusts Node)
    Vikram->>Node: Key Exchange (Encrypted)
    Note over Vikram, Node: Tunnel Established. Surveillance Blinded.
```

**Step 3: The Command**

Vikram smiled. "The tunnel is secure. The Overseer is blind. It sees traffic, but only static."

He typed the command to the Node:
`> INITIATE_TRANSFER(1,000,000,000 credits TO destination_wallet)`
`> USER: Anaya_Rao` (Using Anaya's stolen credentials)

The command flew through the encrypted tunnel, bypassing the firewalls, and landed directly in the Node's kernel inside the Mainframe.

The Node executed the request against the Syndicate's Core Banking System.

---

## Epilogue: The Glitch

The screen flashed green. **AUTHENTICATION SUCCESSFUL.**
*User 'Anaya_Rao' verified.*

Vikram let out a breath. "We did it."

Then the screen turned red.

**ERROR: AUTHORIZATION FAILED.**
**User 'Anaya_Rao' belongs to group 'CITIZEN'. Required group: 'SYNDICATE_ADMIN'.**
**Transaction Denied.**

Vikram froze. They had spent weeks building a perfect, unbreakable tunnel. They had verified identities. They had encrypted the data. They had blinded the surveillance.

But they had forgotten to check if Anaya's account actually had *permission* to move the money.

**"ALERT,"** the Mainframe boomed, its voice echoing across the rooftop. **"UNAUTHORIZED ADMIN ACCESS ATTEMPT DETECTED. USER TRIANGULATED."**

Because they had authenticated so perfectly, the Mainframe knew *exactly* who was hacking it.

"Run," Vikram whispered.

This time, the encryption couldn't save them.

---

## Chapter 7: The Hard Reset (AuthZ vs AuthN)

*The sewers beneath Budwar Peth. 03:00 Hours.*

Vikram stumbled, his heavy carbon-fiber arm sparking against the damp brickwork. Smoke still rose from his shoulder joint—a souvenir from the Syndicate's Hunter-Seeker drone that had blown apart their rooftop position.

Anaya dragged him into a maintenance alcove. "Stop. We need to stabilize that servo."

"We lost the Faraday Cafe," Vikram gritted out, the pain evident in his voice. "Rojan is gone. The equipment is slag. And for what? We *had* the connection. We *had* the tunnel."

"We had **Authentication**," Anaya corrected, ripping a piece of her sleeve to bind his arm. "We proved *who* we were. But we didn't prove *what* we were allowed to do."

Vikram slumped against the wall. "I don't get it. We had the ID card. The Certificate."

"Imagine a club," Anaya said, her voice echoing in the tunnel. "The bouncer at the door checks your ID. That's **Authentication (AuthN)**. You're Vikram. You're allowed inside."

"Okay..."

"But inside the club, there's a VIP area. The bouncer at *that* rope doesn't care who you are. He cares if you're on the *list*. That's **Authorization (AuthZ)**. The Mainframe knew we were 'Anaya_Rao'. But 'Anaya_Rao' isn't on the list for 'Transfer 1 Billion'."

"So we failed because we didn't check the guest list," Vikram spat.

"It's worse," Anaya said, pulling up a schematic on her cracked wrist-comp. "The Mainframe is **Stateless**. It doesn't keep a guest list at the door."

---

## Chapter 8: The Golden Ticket (JWT)

"To understand how to beat them," Anaya explained, "You have to understand how they think. The Syndicate uses **Stateless Architecture**."

 **The Ledger vs. The Wristband**

"In the old days (Stateful)," Anaya began, projecting a hologram into the sludge, "The server kept a ledger (Session ID). Every time you asked for something, it checked its book."

*   **Stateful:** 
    *   Client: "I'm Session #99."
    *   Server: *Checks memory.* "Ah, #99 is Anaya. She is logged in."

"But the Syndicate is too big for ledgers. They have thousands of servers. They can't sync a ledger across all of them fast enough. So they use **Tokens**."

*   **Stateless:**
    *   Client: "Here is my Token (Wristband). It says I am Anaya and I am Admin."
    *   Server: *Reads Token.* "Signed by the Boss? Okay, come in."

"This token," Anaya pointed to a jagged string of characters, "is a **JSON Web Token (JWT)**."

It looked like three colors of noise separated by dots:
`Header.Payload.Signature`

**1. The Header (Red):** "Says 'I am a JWT signed with RS256'."
**2. The Payload (Purple):** "The data. 'User: Rajan', 'Role: Syndicate_Admin'."
**3. The Signature (Blue):** "The wax seal. Proof that the Syndicate itself created this token."

Vikram's eyes narrowed. "So we don't need to hack the database to change our permissions. We just need to show up wearing the right wristband."

"Exactly," Anaya smiled, a dangerous glint in her eyes. "We can't forge a signature—we don't have their private key. But we don't *need* to forge it."

"We just need to steal one from someone who already has it."

---

## Chapter 9: The Replay

*The Skywalk, Deccan Gymkhana. 06:00 Hours.*

The target was Lieutenant Kael, a mid-level Syndicate enforcer known for his arrogance—and his high-clearance access. He sat at a high-end terminal, sipping synthetic coffee, managing drone deployments.

"He's transmitting," Anaya whispered from the shadows. "He's authorizing drone repairs. That requires Admin privileges."

Vikram aimed his sniffers not at the encrypted tunnel (which they couldn't break), but at the **endpoint** where Kael's personal link connected to the local mesh. It was a momentary lapse in Kael's opsec.

**Sniffer Output:**

```http
POST /api/drones/deploy HTTP/1.1
Host: api.syndicate.net
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkthZWwiLCJyb2xlIjoic3luZGljYXRlX2FkbWluIiwiaWF0IjoxNTE2MjM5MDIyfQ.Sw5...[truncated]...
```

"Got it," Vikram hissed. "The **Bearer Token**."

"It's valid for 15 minutes," Anaya said, her fingers flying across her deck. "If we use it, the server won't know it's *us* using it. It just sees a valid wristband."

**The Attack:**

Anaya crafted a new request. She didn't use her credentials. She didn't try to log in. She simply attached the stolen wristband to her command.

```bash
curl -X POST https://api.syndicate.net/v1/transfer \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIs..." \
  -d '{"amount": 1000000000, "to": "Resist_Wallet_77"}'
```

**Understanding the Vulnerability:**
Because the server is **Stateless**, it doesn't check *where* the request came from. It doesn't check if Kael is actually sitting at that computer. It only checks:
1.  Is the Signature valid? **YES.** (It was signed by the Syndicate).
2.  Is the Token expired? **NO.** (It has 14 minutes left).
3.  Does the Token have the 'admin' role? **YES.**

**The Result:**

The screen blinked.

**> AUTHORIZATION SUCCESSFUL.**
**> TRANSFER COMPLETE.**

Vikram watched the credits drain from the Syndicate's account. "They trusted the token more than the user."

"That," Anaya said, closing her laptop as sirens began to wail in the distance, "is the Achilles' Heel of statelessness. If you lose your keys, you change the lock. But if you lose your token... anyone can be you."

They vanished into the neon rain, 1 billion credits richer—and a hell of a lot wiser.

**Network secured. Authorization granted.**

---

> **The End?** 
> *Security is never finished. It is only improved.*
