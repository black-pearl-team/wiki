---
title: HTTP1.x, HTTP2.0 and HTTPS, No More Mixing Them Up
description: 
published: true
date: 2026-09-30T13:35:19.000Z
tags: network, http, https
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/network/http.md)

# Main differences between HTTP1.0 and HTTP1.1

## Persistent connections
HTTP1.1 supports persistent connections (PersistentConnection) and request pipelining (Pipelining): multiple HTTP requests and responses can be sent over one TCP connection, which cuts the cost and latency of opening and closing connections. HTTP1.1 turns on Connection: keep-alive by default, which to some extent makes up for HTTP1.0's drawback of creating a connection for every request.
Reference: [Differences between HTTP1.0, HTTP1.1 and HTTP2.0](https://juejin.cn/post/6844903489596833800) (in Chinese)

## How does HTTP1.1 deal with HTTP's head-of-line blocking problem?
[The HTTP head-of-line blocking problem] HTTP transfer follows a request-response model: messages must go one sent, one received, and the tasks inside are put in a [task queue] and run one after another. Once the request at the head of the queue is processed too slowly, it blocks the processing of the requests behind it.
  1. **Concurrent connections**
  With [one domain allowed to open several persistent connections], it's like adding task queues, so the tasks in one queue don't block all the others. RFC2616 once said a client could have at most 2 concurrent connections, but in current browser standards the limit is actually much higher: 6 in Chrome.
  > But this doesn't really solve the problem at the level of HTTP itself; it just adds TCP connections to spread the risk. It has downsides too: several TCP connections compete for limited bandwidth, so the requests that truly have high priority can't be handled first.
  2. **Domain sharding**
  Can't one domain have 6 concurrent persistent connections? Then I'll just split off a few more domains. For example, under the `google.com` domain you can split off lots of second-level domains that all point to the same server, so more persistent connections can run concurrently, and in fact it handles head-of-line blocking better too.

---

# How HTTP2.0 improves on HTTP1.x (performance)
2.0 is mainly about performance; HTTPS already does a very good job on security

## 1. Header compression

In the days of HTTP/1.1 and before, the request body usually went through a corresponding compression and encoding step, specified by the `Content-Encoding` header field. But what about compressing the header fields themselves?
HTTP1.x headers carry lots of information, and they're sent again every time (when the request fields are very complex, especially for GET requests, the request message is almost all headers). For header fields, HTTP/2 also adopts a matching compression algorithm, **HPACK**, to compress the request headers.

HPACK is designed specially for HTTP/2, and it has two main highlights:
- First, a [hash table] is built between the server and the client, and the fields in use are stored in this table. Then, when a value that has appeared before is transmitted, only its index (like 0, 1, 2, ...) needs to be sent to the other side, and the other side just looks it up in the table with that index. Passing indexes like this lets request header fields be trimmed down and reused to a very large degree.
<img src="/technology/network/http/compressheader.png" alt="compressheader.png" width="70%">

    > Tip
    HTTP/2 does away with the concept of the start line, and turns the request method, URI and status code in the start line into header fields; these fields all carry a ":" prefix to tell them apart from other request headers.
- Second, [Huffman coding for integers and strings]. The idea of Huffman coding is to first build an index table of all the characters that appear, then make the indexes of the most frequent characters as short as possible; what gets transmitted is a sequence of these indexes, which can reach a very high compression rate.

## 2. Multiplexing
Above we mentioned how HTTP1.1 optimizes its way around head-of-line blocking, and saw that it doesn't solve it at the HTTP application layer but leans on what the TCP transport layer can do. HTTP2.0, then, solves head-of-line blocking in the HTTP protocol itself.

**The new binary format (Binary Format) [binary framing]**

  In the HTTP1.x protocol, messages (mainly the headers) don't use binary data but **text form**. (Text can take many forms: text, images, video, any kind of data)

  HTTP/2 finds plaintext transfer too much trouble for machines and awkward for computers to parse, because text has ambiguous characters, e.g. is a carriage return/line feed content or a separator? Internally a state machine is needed to tell, which is fairly inefficient. So HTTP/2 simply turns every message into a binary format and transmits strings of 0s and 1s all the way, which makes parsing easy for machines.

  The old `Headers + Body` message format is now split into individual binary **frames**, with `Headers` frames holding the header fields and `Data` frames holding the request body data. After framing, what the server sees is no longer complete HTTP request messages one by one, but a pile of out-of-order binary frames. These binary frames have no order between them, so they don't queue up and wait, and HTTP's head-of-line blocking problem is gone.

Both sides of the communication can send binary frames to each other, and this **bidirectional sequence of binary frames** is also called a **stream (Stream)**. HTTP/2 uses streams to carry the communication of multiple data frames over one TCP connection, which is the concept of [multiplexing].

> **If they're sent and received out of order, how are these out-of-order data frames handled in the end?**
First, to be clear: "out of order" means streams with different IDs are out of order, but frames with the same Stream ID are always transmitted in order. When the binary frames arrive, the other side assembles the frames with the same Stream ID into complete request and response messages.

## 3. Setting request priority
The structure of a frame transmitted in HTTP/2 is shown below:
<img src="/technology/network/http/binarystream.png" alt="binarystream.png" width="65%">
Each frame is split into a **frame header** and a **frame body**. First comes a three-byte frame length, which is the length of the frame body.

Then comes the **frame type**, which roughly splits into data frames and control frames.
Data frames hold HTTP messages, and control frames manage the transmission of streams. [(For example, using a PRIORITY frame to change a stream's priority.)]

The next byte is the **frame flags**, with 8 flag bits in total; common ones are `END_HEADERS`, meaning the header data ends, and `END_STREAM`, meaning sending in one direction ends.

The last 4 bytes are the `Stream ID`, the **stream identifier**. With it, the receiver can pick out the frames with the same ID from the out-of-order binary frames and assemble them in order into request/response messages.

Take an ordinary request-response process as an example:
<img src="/technology/network/http/request&amp;response.png" alt="request&amp;response.png" width="50%">
At first both sides are idle. When the client sends a Headers frame, a `Stream ID` starts being allocated and the client's stream opens; after the server receives it, the server's stream opens too. Once the streams on both ends are open, they can pass data frames and control frames to each other.

When the client wants to close, it sends an `END_STREAM` frame to the server and enters the half-closed state; now the client can only receive data, not send it.

After receiving this `END_STREAM` frame, the server also enters the half-closed state, but on the server's side it can only send data, not receive it. Then the server also sends an `END_STREAM` frame to the client, meaning the data has all been sent, and both sides enter the closed state.

If a new stream is to be opened next time, the stream ID has to increase, up to the limit; once the limit is reached, a new TCP connection is opened and counting starts over. Since the stream ID field is 4 bytes long and the highest bit is reserved, the range is 0 to 2 to the 31st power, about 2.1 billion.

Characteristics of stream transmission:
  - Concurrency. Several frames can be sent at the same time over one HTTP/2 connection, unlike HTTP/1. This is also the basis for multiplexing.
  - Incrementing. Stream IDs can't be reused; they go up in order, and once they reach the limit a new TCP connection starts over from the beginning.
  - Bidirectionality. Both the client and the server can create streams without getting in each other's way, and either side can be the sender or the receiver.
  - Settable priority. Data frames can be given a priority, so the server handles important resources first and the user experience improves.


## 4. Server Push
In HTTP/2, the server is no longer completely passive, only receiving and answering requests; it can also create new streams to send messages to the client. Once the TCP connection is set up, if, say, the browser requests an HTML file, the server can return, along with the HTML, the other resource files referenced in that HTML, cutting down the client's wait.

---

# HTTPS

HTTPS isn't a new application-layer protocol. It just replaces part of HTTP's communication interface with the SSL and TLS protocols.

SSL is the **Secure Sockets Layer**, sitting at the session layer (layer 5) of the OSI seven-layer model.
SSL went through three major versions, and only when it reached its third major version was it standardized, becoming **TLS (Transport Layer Security)**. The mainstream version now is TLS/1.2.

Compared with the HTTP protocol, the HTTPS protocol has these extra advantages:
- Authentication: a third party can't forge the server's (client's) identity
- Data privacy: content is symmetrically encrypted, and each connection generates a unique encryption key
- Data integrity: the content goes through integrity checks in transit

**HTTPS is really just HTTP wearing an outer shell of the SSL protocol.**
Once SSL is in use, HTTP gets HTTPS's encryption, certificates and integrity protection.

<img src="/technology/network/http/https.png" alt="https.png" width="50%">

What TLS/SSL does relies mainly on three kinds of basic algorithms:
- Asymmetric encryption: for authentication and key negotiation
- Symmetric encryption: encrypting data with the negotiated key
- Hash functions: verifying the integrity of information

## Solving possible eavesdropping on content: encryption
Asymmetric encryption is used for the key exchange, and symmetric encryption for the later stage of setting up communication and exchanging messages.

In practice:
The side sending the ciphertext encrypts the "symmetric key" with the other side's public key, and the other side decrypts it with its own private key to get the "symmetric key". This way, on the premise that the exchanged key is safe, the two communicate with symmetric encryption. So HTTPS uses a hybrid encryption scheme that uses symmetric and asymmetric encryption together.

## Solving possible tampering with messages: digital signatures
Network transmission passes through many intermediate nodes. Although the data can't be decrypted, it could be tampered with, so how do we verify its integrity? ---- By checking the digital signature.

Digital signatures do two things:
- They confirm that a message really was signed and sent by the sender, because nobody else can fake the sender's signature.
- A digital signature can confirm a message's integrity, proving whether or not the data has been tampered with.

    **How a digital signature is generated:**
    <img src="/technology/network/http/generatesignature.png" alt="generatesignature.png" width="85%">

    Sender: take a piece of text, first **generate a message digest with a Hash function**, then **encrypt it with the sender's private key to produce the digital signature**, and send it to the receiver together with the original text.

    **How a digital signature is checked:**
    <img src="/technology/network/http/checksignature.png" alt="checksignature.png" width="85%">

    Receiver: the receiver can **decrypt the encrypted digest only with the sender's public key**, then **produces a digest of the received original text with the HASH function** and compares it with the digest from the previous step. If they match, the received information is complete and wasn't modified in transit; otherwise it was modified, so a digital signature can verify the integrity of information.

## Solving possible impersonation of the communicating parties: digital certificates

Certificate Authority (CA for short)
A certificate authority stands in the position of a third party that both the client and the server can trust.

How a certificate authority does business:
- The server's operator submits the public key, organization info, personal info (domain) and so on to the third-party CA and applies for certification;
- The CA verifies that the information the applicant provided is true through all kinds of online and offline means, e.g. whether the organization exists, whether the business is legal, whether they own the domain, and so on;
- If the information passes review, the CA issues the applicant a certification file, the **certificate**.
The certificate contains the plaintext of the following: **the applicant's public key**, the applicant's organization and personal info, **information about the issuing CA**, the validity period, the certificate serial number and so on, and it also contains a **signature**.
How the signature is produced: first, a **hash function** computes the **message digest** of the public plaintext; then the CA's private key encrypts the digest, and the ciphertext is the **signature**;
- When the Client sends a request to the Server, the Server returns the certificate file;
- The Client reads the relevant plaintext in the certificate, computes the message digest with the same hash function, then decrypts the signature data with the corresponding CA's public key and compares it with the certificate's **message digest**. If they match, the certificate is confirmed as legitimate, i.e. the server's public key can be trusted.
(The certificate authorities' public keys are built into the browser ahead of time.)
- The client also verifies the certificate's domain info, validity period and so on; the client has the certificate info (including public keys) of trusted CAs built in, and if a CA isn't trusted, its certificate can't be found and the certificate is judged illegitimate as well.
<img src="/technology/network/http/certificateauthority.png" alt="certificateauthority.png" width="75%">


Reference: [An in-depth look at how HTTPS works](https://github.com/ljianshu/Blog/issues/50) (in Chinese)

Finally:

![httpsflow.png](/technology/network/http/httpsflow.png)


# Summary
- HTTPS optimizes HTTP for security
- HTTP2.0 optimizes HTTP for performance

<img src="/technology/network/http/summary.png" alt="summary.png" width="65%">
