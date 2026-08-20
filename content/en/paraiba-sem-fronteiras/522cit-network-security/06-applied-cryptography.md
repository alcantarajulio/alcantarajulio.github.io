---
title: "Applied cryptography"
description: "Confidentiality, integrity and authenticity: symmetric ciphers, public keys, digests, HMAC, signatures and PKI."
date: 2026-08-19
chapter: 6
translationKey: "psf-522-06"
weight: 60
tags: ["cryptography", "openssl", "pki", "hmac", "aes"]
---

Earlier chapters used cryptography without explaining it. SSH encrypts the session, OSPF
authenticates the adjacency with a digest, SNMPv3 protects the payload. This chapter
opens the box, because the next one builds an entire VPN on top of it.

## Four goals that get confused

Confidentiality stops anyone reading the message. Integrity lets you detect that someone
changed it. Authenticity proves who sent it. Non-repudiation stops the sender denying
afterwards that they sent it.

No single tool delivers all four, and picking the wrong one produces a system that looks
secure. Encrypting without integrity checking lets an attacker modify the ciphertext
blindly and cause an effect at the receiver. Checking integrity without authentication
lets an attacker alter the message and recompute the digest. Each goal has its own tool.

## The principle that defines the field

Kerckhoffs, in the nineteenth century, established that a system must stay secure even
when everything about it is known except the key. This sounds obvious today and still
gets violated whenever someone trusts a secret algorithm.

The reason is practical. Algorithms leak: through reverse engineering, through an
employee leaving, through accidental publication. A key gets replaced in minutes. A
system that depends on the secrecy of its algorithm loses everything at once, with no
path to recovery.

## Symmetric ciphers

A symmetric cipher uses the same key to encrypt and decrypt. It is fast and handles
volume, which is why it protects the body of every encrypted session you use.

DES, with a 56-bit key, fell to brute force in 1998 and belongs in no new design. 3DES
applied the same cipher three times to buy time, costs heavily in performance, and is
being retired. AES is the current standard, with keys of 128, 192 or 256 bits.

The mode of operation matters as much as the algorithm. The simplest mode encrypts each
block independently, so identical plaintext blocks produce identical ciphertext blocks,
and an encrypted image stays recognisable. Block chaining fixes that by mixing each
block with the previous one. Modern authenticated modes deliver encryption and integrity
checking in the same operation, which is the right answer for almost any new case.

The problem symmetric cryptography cannot solve alone is key distribution. Two parties
need to agree on a secret before talking, and agreeing on a secret over an insecure
channel is the original problem.

## Public keys

Asymmetric cryptography uses a pair of keys joined by mathematics. What one encrypts,
only the other decrypts. You publish one and keep the other.

That solves distribution, and more. Encrypting with the recipient's public key gives
confidentiality, since only they decrypt. Encrypting with your private key gives
authenticity, since anyone verifies with your public key and only you could have
produced it.

The cost is performance. An asymmetric operation runs orders of magnitude slower than a
symmetric one, which rules out encrypting a whole stream with it. Hence the hybrid
design every modern protocol uses: asymmetric negotiates a session key, symmetric
encrypts the data with it.

Diffie-Hellman key exchange deserves attention because it does something
counterintuitive: two parties reach a shared secret by exchanging messages in public,
while an eavesdropper cannot derive the same secret. The IPsec in the next chapter
depends on it.

Among the families, RSA bases its security on the difficulty of factoring large numbers
and needs 2048 bits or more. Elliptic curve cryptography reaches equivalent strength
with a far smaller key, which matters on low-power devices.

## Cryptographic digests

A hash function turns input of any size into fixed-size output, so that the same input
always produces the same output and changing one bit changes the whole output.

It is not encryption, because no inverse operation exists, and that confusion turns up
in exams. A digest detects modification; it hides nothing.

MD5 and SHA-1 suffer practical collision attacks, meaning someone can produce two
different documents with the same digest. For integrity checking against an adversary,
use SHA-2 or SHA-3.

Plain hashing hits a limit. If an attacker alters the message, they recompute the digest
and send both. The defence mixes a key into the calculation, and the result is HMAC:
without the key, nobody produces a valid digest. This is what the OSPF and NTP from
earlier chapters use.

## Signatures and public key infrastructure

A digital signature combines a digest and a private key. You compute the digest of the
message and encrypt it with your private key. The receiver decrypts with your public
key, recomputes the digest, and compares. That delivers integrity, authenticity and
non-repudiation at once, and costs little, since the slow operation runs over the digest
rather than the message.

One piece is missing. Verifying a signature with a public key proves that the signer
holds the matching private key, and proves nothing about who that person is. The public
key you received might belong to the attacker.

Public key infrastructure solves this by delegating trust. A certificate authority
verifies someone's identity and issues a certificate, which is that person's public key
signed by the authority. You trust the authority, so you accept the keys it vouches for.

This moves the problem rather than removing it, and the move pays off because the number
of authorities is small. The system carries known costs: a compromised authority
invalidates every certificate it issued, and revocation depends on lists or real-time
queries that not every client checks.

## Practice

This practice runs on Linux with OpenSSL rather than Packet Tracer, because the
simulator implements none of these operations. Any machine with OpenSSL installed works.

### Digests and the avalanche effect

```text
$ echo -n "network security" | openssl dgst -sha256
$ echo -n "network securiti" | openssl dgst -sha256
```

Compare the two outputs. One character of difference changes the entire digest, and that
is what makes integrity checking useful.

### HMAC against a plain digest

```text
$ echo -n "transfer 100" | openssl dgst -sha256
$ echo -n "transfer 100" | openssl dgst -sha256 -hmac "secret-key"
```

Ask the student to recompute the first with no extra information, which they can, and
then the second, which they cannot without the key.

### Symmetric encryption

```text
$ openssl enc -aes-256-cbc -pbkdf2 -salt -in message.txt -out message.enc
$ openssl enc -d -aes-256-cbc -pbkdf2 -in message.enc -out recovered.txt
$ diff message.txt recovered.txt
```

### Key pair, signature and verification

```text
$ openssl genrsa -out private.pem 2048
$ openssl rsa -in private.pem -pubout -out public.pem
$ openssl dgst -sha256 -sign private.pem -out signature.bin message.txt
$ openssl dgst -sha256 -verify public.pem -signature signature.bin message.txt
```

Change one byte of `message.txt` and verify again. Verification fails, and that failure
is the demonstration of integrity.

### Certificates

```text
$ openssl req -new -key private.pem -out request.csr
$ openssl x509 -req -days 365 -in request.csr -signkey private.pem -out certificate.crt
$ openssl x509 -in certificate.crt -text -noout
```

The certificate you just built is self-signed, so no browser trusts it. That is the
chapter's demonstration: the mathematics works and the trust is missing, and filling
that exact gap is what a certificate authority does.

### On the router side

```text
R1(config)# crypto key generate rsa general-keys modulus 2048 label CA-KEYS
R1(config)# crypto pki trustpoint CA-LOCAL
R1(ca-trustpoint)# enrollment terminal
R1(ca-trustpoint)# subject-name CN=R1.redes.ufcg.br
R1(ca-trustpoint)# revocation-check none
R1# show crypto key mypubkey rsa
R1# show crypto pki certificates
```

## Course labs

The subject appears in lab 15.0.3 of module 15, and in 16.3.10 and 16.3.12 of module 16.
Lab 16.3.12 closes the circle with chapter 1: you capture a Telnet session and an SSH
session in Wireshark and compare what shows up.

The next chapter uses all of this to build a tunnel between two routers.
