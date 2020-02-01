---
title: "Possible improvement for Dgraph Badger encryption at rest"
date: 2020-01-31T20:22:55-08:00
draft: true
---

# How Badge DB protect data at rest

This is how Badge DB protect client's data at rest, I copied from Dgraph blog https://blog.dgraph.io/post/releasing-badger-v2/


> With a key focus on security, Badger now provides an option to encrypt its data!
To use encryption, you need to provide Badger an encryption key using the Options.WithEncryptionKey API.
Badger uses a different key to encrypt the data, these are called data keys, and they are auto-generated.
The data key expires at regular intervals, and Badger auto-generates a new one every time it expires. The new data will be encrypted using the new data key, while the rest of the data still be encrypted with the data keys generated before. The expiry time for the data key is configurable using the Options.WithEncryptionKeyRotationDuration function.
Badger stores the history of the data keys generated. All of them are encrypted using the encryption key, which you provided initially.
The two-step encryption process, by design, provides an additional layer of security by allowing you to rotate keys frequently without having to re-encrypt your dataset.


We can see Badge DB is using rotate keys to protect data at rest. This is a great idea. However, I can still find some improvements.

# What if the server is hacked?

The Badge DB generates all encrypted keys on the server-side.  I assume the keys are stored somewhere in the server too. This method is still OK because unless hackers can get the golden key from the customer, there is no way to decrypt those encryption keys.  My only concern is in the case of hacker has hacked into the server and access the memory contains the customer's golden key. All encryption keys would be decrypted no matter how many rotate keys we were using. 

In confidential computing, we have to assume the server's OS can be hacked. Actually, it does happen frequently. 

# Put the golden key and decryption process inside TEE(Trusted Execution Environment).

It is very hard to put all tech stack under protection due to the size of the codebase (TCB, Trusted CodeBase). A practical solution is to protect the most sensitive code into SGX, using CPU's hardware protection feature. In our Badge DB case, the decryption of keys using the golden key is the most sensitive code. Let's assume we are using Intel SGX as our TEE. Due to the limitation of Intel SGX, we try not to load all the code into SGX. The majority code is still running in the none-TEE world, which is the so-called untrusted space. Before Badge DB decrypts the rotate keys, it will call functions in the trusted world inside TEE. Those function and the memory it located inside TEE is not visible from the outside of TEE. Even if the OS is compromised, the memory located by the TEE is still safe. The golden key should ONLY live inside TEE. The function will ask the customer to TLS the golden key into the TEE directly using CPU's key pairs. Since no one else knows the CPU's key pair rather than CPU itself, it is safe to send the golden key TLS to the CPU's TEE. Of course, the customer should verify the request to make sure the request is actually from the CPU's TEE by using TEE's remote attestation. Once the request passes verification, the customer will send the golden key to TEE directly. In this case, no one can get the plain text of the golden key, even the OS. Once the TEE received the golden key, decrypt it to clear. Use this golden key to decode other rotate keys. Because we cannot do all our Badge DB work inside TEE, the "active" rotate key still needs to send out of the TEE and run inside the untrusted space. The good part is that even a particular active key leaks, only a small amount of data at risk, the golden key is still safe. 

# Balance between performance and security

Ideally, we should put all code related to rotation keys into the TEE, so that even OS is compromised, the hacker cannot get a single rotate key. In fact, this is not practical. Every time when code inside TEE wants to access anything outside of TEE, the data need to run an encryption-decryption process. This is a big performance penalty. If we try to put everything inside the TEE to minimize the traffic between TEE and the outside world, the TCB to be protected will be too big to be safe. The larger codebase is more volnerable than smaller codebase. 

# One step further, decentralize the remote attestation network

Remote attestation should run inside in clients' trust machine. By doing so, clients do not need to trust any outside resource to protect their golden keys. However, remote attestation is not a piece of cake. Usually, clients would like to trust a remote attestation service, and very likely, a centralized service. This comes to another problem: Can we trust this centralize service?   No doubt the centralized server will be heavily guarded, but still a big bait to hackers due to the huge benefits. DDoS is also some kind of attack hard to prevent. If we decentralized the remote attestation service to thousands of individual nodes, it would be much harder for attackers. At least the DDoS cost to thousands of nodes is much higher than a single service. 

The idea of building a decentralized remote attestation network on the blockchain is in my previous blog posts. 




