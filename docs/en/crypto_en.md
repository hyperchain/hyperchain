## Cryptography Algorithm

### 1.Overview

Hyperchain blockchain platform to support multi-level cryptography encryption to ensure data security, the use of the following cryptographic algorithms to ensure data security issues:

- Elliptic curve digital signature:secp256k1,secp256r1
- Symmetric encryption algorithm:3DES,AES
- Key exchange:ECDH
- Hash:Keccak-256

### 2. Elliptic curve digital signature

Elliptic Curve Digital Signature Algorithm (ECDSA) is a simulation of Digital Signature Algorithm (DSA) using [Elliptic Curve Cryptography (ECC). ECDSA became ANSI standard in 1999 and became the IEEE and NIST standards in 2000. It was accepted by the ISO in 1998 and some of the other standards that include it are also under ISO's consideration. Unlike the discrete logarithm problem (DLP) and the integer factorization problem (IFP), the elliptic curve discrete logarithm problem (ECDLP) has no solution to the subexponential time. Therefore, the unit bit strength of elliptic curve cryptography is higher than that of other public key systems. Elliptic curve graphics as shown below:

![](../../images/ecdsa.gif)

The Hyperchain blockchain uses the secp256k1 curve to sign and verify the platform transaction to ensure the correctness and completeness of the transaction. At the same time, the platform supports the use of secp256r1 curve to sign messages between nodes to verify the integrity and correctness of message communication between nodes. Node message signature is pluggable, you can open it through the configuration file.the configuration file in the namespace.toml:

```
[encryption.check]
sign = true  #enable Signature
```

That is, when ```sign = true```, you need to verify the signature of messages between nodes, otherwise you do not need to verify.

### 3.Symmetric encryption algorithm

The Hyperchain blockchain platform supports both AES and 3DES symmetric encryption algorithms to ensure ciphertext transfer between nodes.

3DES, also known as Triple DES, is a mode of the DES encryption algorithm that uses three 56-bit keys to encrypt 3DES data three times. The Data Encryption Standard (DES) is a well-established encryption standard in the United States that uses symmetric key cryptography. DES uses a 56-bit key and cryptographic block method, whereas in the cryptographic block method, the text is divided into 64 Bit-sized text blocks are then encrypted. 3DES is safer than the original DES. Encryption algorithm, which is specifically implemented as follows: Let Ek () and Dk () represent the encryption and decryption process of the DES algorithm, K represents the key used by the DES algorithm, M represents the plaintext, and C represents the ciphertext:

```
3DES Encryption Process：C=Ek3(Dk2(Ek1(M)))
3DES Decryption Process：M=Dk1(EK2(Dk3(C)))
```

AES, also known as Advanced Encryption Standard, is a block encryption standard used by the U.S. federal government. The AES algorithm is based on permutation and permutation operations, and AES uses several different methods to perform permutation and permutation operations. AES is an iterative, symmetric-key-group password that uses 128, 192, and 256-bit keys and encrypts and decrypts data with 128-bit (16-byte) packets. Unlike public key password use key pairs, symmetric key passwords use the same key to encrypt and decrypt data. The encrypted data returned by the block password has the same number of bits as the input data. Iterative encryption uses a loop structure in which the input data is repeatedly replaced and replaced.

In addition,Hyperchain blockchain platform supports symmetric encryption algorithm configuration option to encrypt node messages, configured under namespace.toml:

```
[encryption.security]
algo = "3des"   # Selective symmetric encryption algorithm (pure,3des or aes)
```

Support pure, 3des and aes three parameters:

- pure:do not perform any encryption, in plain text
- 3des:3des encryption and decryption
- aes:aes encryption and decryption

### 4.Key exchange algorithm

ECDH is a combination of ECC and DH algorithms for key negotiation. The exchange parties can negotiate a key without sharing any secrets. ECC is a cryptosystem based on the discrete logarithm problem of elliptic curve. Given a point P and an integer k on the elliptic curve, it is easy to solve Q = kP. Given a point P, Q, we know that Q = kP , Find the integer k is indeed a problem. ECDH is built on this math puzzle.

In addition, Hyperchain exchanges keys when the node handshake for the first time and generates shared keys for each other. This key is the symmetric encryption key between the nodes.

### 5.Hash algorithm

Hash algorithm, Hyperchain platform using Keccak256 algorithm for Hash calculation, the results of the settlement for the signature of the summary, but also can be used for address calculation.