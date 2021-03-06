# Q1 9.4

> Design a protocol based on Protocol 1 in Section 9.3.2 which, in addition to the existing protocol goals, also allows Alice to confirm Bob is still alive.

Considering the original protocol is meant to indicate that Alice is alive. We can simply double the protocol, and maybe try to truncate it a little bit for efficiency.

A naive approach would be to simply "double" the protocol:

- Bob -> Alice
  - $r_b ||$ It's Bob, are you OK?
- Alice -> Bob
  - $r_a || r_b ||$ Bob $||$ Yes, I'm OK. You? + $\text{MAC}_K($r_b ||$ Bob $||\$ Yes, I'm OK, You?)
- Bob -> Alice
  - $r_a ||$ Alice $||$ Glad we're together + $\text{MAC}_K(r_a ||$ Alice $||$ Glad we're together)

> **Note**: We could have based this a bit more off of Protocol 6 and assumed that Bob can generate timestamps that Alice can verify, and then we could have done this in 2 steps, but we would have needed to change the assumptions and thus the framework.

Analysis:

The analysis is identical up to and through the 2nd message - the only addition in that message is the $r_a$, but since it is in the MAC and also not, Bob still has enough information to confirm the MAC.

- _Data Origin Authentication of Bob's reply_
  - Same analysis for Alice's reply - since the message is MACed, but the assumption that only Bob and Alice have access to the symmetric key $K$, we can guarantee that the message came from Bob.
- _Freshness of Bob's reply_
  - Yes, since it contains Alice's nonce $r_a$
- _Assurance Bob's reply corresponds to Alice's request._
  - Yes, since it contains Alice's nonce $r_a$, and it also contains Alice's identifier, so it is resistance to a reflection attack.

# Q2 9.5

> The security objectives of a cryptographic protocol are for Alice to be able to send a highly confidential email message to Bob in such a way that:
>
> - Bob can prove to Charlie the email came from Alice; and
> - Alice receives confirmation Bob received the email message

1. Determine the protocol goals

- **Confidentiality of Alice's message to Bob**
- **Non-repudiation of Alice's message to Bob**
- **Assurance that Bob received Alice's message**

2. Propose a simple cryptographic protocol (including assumptions, flow, messages, and actions) for achieving these goals.

- Assumptions
  - Alice and Bob have a shared encryption key $K$, and a shared MAC key $M$
  - Alice has a signature key $K_A$ that Bob and Charlie can verify.
  - Alice has a means of generating random nonces.

* Flow

  - Clearly the messages must flow in the order specified by the messages

* Messages
  - Alice -> Bob
    - $E_{K}(M||r_a)$
    - $\text{Sig}_{K_A}(E_K(M||r_a))$
  - Bob -> Alice
    - $r_a ||$ Gotchya
    - $\text{MAC}_M(r_a || \text{Alice} || \text{Gotchya})$

- Actions
  - Upon receiving Alice's first message:
    - Bob decrypts the first component to get $M$ the message and $r_a$ the nonce generated by Alice.
    - Bob can store the signature and the encrypted message. If he ever needs to prove to Charlie that the message came from Alice he can share the signature and the encrypted data.
  - For the reply
    - Bob sends the nonce $r_a$ and some string such as "Gotchya", and also sends a MAC using the shared key he and Alice have under the information $r_a$, Alice's identifier, and the string he sent.
  - When Alice gets this message, she can verify that the message came from Bob by using the MAC, and that it was not tampered (or reflection attack by adding in her identifier between the text and the nonce). Finding the same nonce she began with assures her that Bob was able to decrypt the message and extract the nonce.

# Q3 9.6

> Let $p =23$ and $g=11$ be system wide parameters for the Diffie-Hellman protocol.

1. Show that if Alice chooses $a=2$ and Bob chooses $b=4$, then, by following the Diffie Hellman protocol, both of them establish the same secret value.

- Well first Alice sends $g^a \mod p = 11^2 \mod 23 = 6$ to Bob.
- Bob the calculates $6^b \mod p = 6^4 \mod 23 = 8$ and sends $g^b \mod p = 11^4 \mod 23 = 13$ to Alice
- Alice then calculates $13^a \mod p = 13^2 \mod 23 = 8$

Therefore they both got $8$ at the end of the protocol.

2. What is the largest symmetric key length Alice and Bob could safely derive directly from this shared secret?

The modulus is 23, which is 5 bits, therefore the largest symmetric key they could derive is a 5 bits long.

3. Explain how attacker Fred can conduct a man-in-the-middle attack using the value $f=3$.

If Fred were in the middle, Fred would receive Alice's first message of $6$, and compute $6^3 \mod 23 = 9$ and send $11^3 \mod 23 = 20$ back to Alice to productively get a shared key of $9$ with Alice ($20^2 \mod 23 = 9$).

Fred would also begin the protocol with Bob, as though he were Alice by sending Bob $20$ to Bob, who would compute $20^4 \mod 23 = 12$ and Fred would ultimately get a shared key with Bob also of value $12$.

# Q4 10.2

> Guidance on key lengths changes over time

1. Name two reputable sources for key length recommendations (other than the ENCRYPT recommendations of Table 10.1), and explain why they are credible sources.

   - NIST, the national institute for science and technology. They are reputable because of their public nature, and because of the industry/academic professionals who engage in their discourse and evaluation.
   - BSI - British Standards Institution, for the same reason as above, they establish federal standards.

We would expect only the greatest level of security required by these two organizations, as they are required to provide standards cryptography across the national landscape.

1. To what extend do their recommendations for symmetric key lengths 'match' the recommendations shown in table 10.1?

   - BSI suggests for 2020 a symmetric key length of 128 bits, which is much larger than the suggestions by ENCRYPT.
   - NIST suggests the same for protection till 2030, 128 bits.
   - For reference, the NSA, which has a MUCH higher standard, suggests 256 bits for symmetric keys.

   - Since the publication of this textbook though, ENCRYPT has also increased it's size for medium term-protection till 2030 to 128 bits. (it was 112 in 2010 by the textbook)

   - Source: KeyLength.com - https://www.keylength.com/en/compare/

1. Given a public-key cryptosystem, explain how experts might determine which key length for this algorithm is 'equivalent' to a symmetric key length of 128 bits.

   - It completely depends on the public-key crypto system, and the perceived hardness of the algorithm utilized. For example, if the best attack of a symmetric algorithm is guessing the key, then we want to find a key-length for our public-key cryptosystem that makes the best attack at least as hard as that.

# Q5 10.6

UKPT schemes offer support for key management in special application environments.

1. Which of the phases of the key management lifecycle shown in in figure 10.1 does a UKPT scheme make straightforward?

   - THE UKPT scheme makes Key storage very straight forward. Since the keys are only used once, they don't need to be persistently stored.

2. Compare the impacts of the Racal and Derived UKPT schemes in the event an attacker compromises a point-of-sale terminal and is able to access any keys stored in the terminal.

   - If an attacker compromises the Racal scheme, they can either cause the terminal to go out of sync with the host, and force it to be decommissioned. Another possibility, since each key register value is computed from the card data, and the transaction data, and the previous value, if the attacker also has access to the initial shared seed, then the attacker can potentially regenerate this entire chain and extract card data from all the previous customers. (this might be hard if they don't know the intermediate keys, but is still potentially possible). In order to get future keys the attacker would need to listen onto card data as well.

   - If an attacker compromises the Derived scheme though, they gain no information regarding past transaction data, but they can calculate all FUTURE keys (since key derivation is only dependent on the counter).

3. Compare the impacts on the Racal and Derived UKPT schemes if there is a communication error in the middle of a transaction.

   - If there is a communication error in the middle of a transaction, and this was not accounted for, it's possible that the Racal scheme will go out of synchronization, causing potential difficulties in recovering previous keys. On the other hand, it's very easy to calculate previous keys with the Derived scheme, so a communication error might be less of a concern.

4. Suggest some key management controls designed to overcome the 'weakness of these two UKPT schemes.

   - Racal suffers from synchronization issues - in order to remove the synchronization problem we could remove the dependency on card data, and add a dependency on a nonce sent by the host. Then each key register is dependent on the previous key register and the nonce from the host. To fix an issue then from a communication delay, the host could keep track of two of the key registers. If the register didn't get updated by the terminal, then it should make sense with the old register, so try and send a new nonce and calculate a new register. If it makes sense with the new register, expire the old one and move on. This maintains the positive property of Racal (and the negation of which that is a downside of Derived) which makes calculating future keys difficult, because an attacker doesn't know what the nonces will be.

# Q6 10.7

Quantum key establishment technology is at a relatively early stage of maturity. Explore the 'state of the art' in quantum key establishment by finding out the following:

1. What is the longest distance over which a symmetric key has been established using QKE?

In 2021, researchers reached 511-km over a optical fibre link.

https://www.nature.com/articles/s41566-021-00848-1#:~:text=A%20'twin%2Dfield'%20repeater,an%20essential%20task%20in%20cryptography.

2. What are the best current data rates?

> Co-existence of the quantum channel with two 800 Gbps and eight 100 Gbps channels was demonstrated for a 70km fiber, with a key rate sufficient to support up to 258 AES-256 encrypted channels at a key refresh rate of 1 key/sec.
>
> https://www.jpmorganchase.com/news-stories/jpmc-toshiba-ciena-build-first-quantum-key-distribution-network

3. Which commercial organizations are selling QKE tech?

Nvidia, QuantLR & LightPath, Toshiba, QuantumCTek, Qasky

- https://finance.yahoo.com/news/quantlr-lightpath-technologies-announce-partnership-200100263.html
- https://www.fierceelectronics.com/embedded/nvidia-continues-quantum-moves-new-qoda-framework
- https://www.prnewswire.com/news-releases/toshiba-safe-quantum-enter-agreement-to-accelerate-quantum-communication-solutions-in-north-america-301580058.html
- https://traveladventurecinema.com/news/100773/qkd-technologies-market-to-witness-huge-growth-by-2022-2028-toshiba-quantumctek-qasky/

The news seems to think there is going to be a HUGE surge in this technology over the next 6 years.

ID Quantique (Geneva), MagiQ Technologies, Inc. (New York), QNu Labs (Bengaluru, India), QuintessenceLabs (Australia), QRate (Russia) and SeQureNet (Paris)

- https://en.wikipedia.org/wiki/Quantum_key_distribution#Commercial

1. Which applications are deploying QKE tech?

- "Geneva to transmit ballot results to the capital in the national election occurring on 21 October 2007"
- "In 2013, Battelle Memorial Institute installed a QKD system built by ID Quantique between their main campus in Columbus, Ohio and their manufacturing facility in nearby Dublin"

* https://en.wikipedia.org/wiki/Quantum_key_distribution#Commercial

# Q7

1. For Protocol 1 and Protocol 3 in Chapter 9, write down (in English prose) the initial assumptions of each party, and what each believes after receiving each message.

   1. Protocol 1
      1. Assumptions
         1. Bob has access to a source of randomness.
         2. Alice and Bob share a symmetric key $K$ only they know.
         3. Alice and Bob agree on the use of MAC algorithm.
      2. After Message #1
         1. Alice believes the message came from Bob.
      3. After Message #2
         1. Bob believes Alice is alive and active.
         2. Bob believes that Alice responded to his message.
   2. Protocol 3 is the same.

2) Translate part A into terms of BAN-style logic. (If BAN doesn't say what you need it to, then define your own modifications)

   1. Assumptions
      1. $B \mid\mkern-3mu\Rightarrow$ a source of randomness
      2. $A  \mid\mkern-3mu\equiv A \leftrightarrow^{k} B$
      3. $B  \mid\mkern-3mu\equiv A \leftrightarrow^{k} B$
      4. $A \leftrightarrow_{MAC} B$ (This means they share an algorithm)
   2. After Message #1
      1. $A \mid\mkern-3mu\equiv (B \mid\mkern-3mu\sim M_1)$
   3. After Message #2

      1. $B \mid\mkern-3mu\equiv (A \mid\mkern-3mu\sim M_2 \land \#(M_2))$
      2. $B \mid\mkern-3mu\equiv M_2 \mid\mkern-3mu\rightarrow B$ This means that B believes M_2 was intended for B.

3. What sort of BAN-style inference rules would make it reasonable for Bob's conclusion at the end of Protocol 1 to be correct?

$$\frac{B \mid\mkern-3mu\equiv \#(M_2) \land B \mid\mkern-3mu\equiv A \mid\mkern-3mu\sim M_2}{B \mid\mkern-3mu\equiv A \mid\mkern-3mu\equiv M_2}$$

4. Why doesn't that work for Protocol 3?

This doesn't work for protocol 3 because we don't have verification that $A \mid\mkern-3mu\sim M_2$. Another way of writing this in this protocol (since Alice and Bob are trustworthy actors), is that $M_2$ _was intended for_ Bob. And we don't get this in protocol 3 because the recipients name is not part of the MAC.
