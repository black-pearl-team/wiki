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

*Much of the Chinese page is compiled from two Chinese articles (linked below); this English page sums the points up in our own words instead of translating them.*

# Main differences between HTTP1.0 and HTTP1.1

## Persistent connections
HTTP1.1 keeps connections open (PersistentConnection) and can pipeline requests (Pipelining), so many requests and responses share one TCP connection instead of paying for a new one each time; Connection: keep-alive is on by default, which fixes HTTP1.0's one-connection-per-request cost.
Reference: [Differences between HTTP1.0, HTTP1.1 and HTTP2.0](https://juejin.cn/post/6844903489596833800) (in Chinese)

## How does HTTP1.1 deal with HTTP's head-of-line blocking problem?
[The HTTP head-of-line blocking problem] Requests and responses go strictly one after another through a [task queue], so one slow request at the front holds up everything behind it.
  1. **Concurrent connections**
  Allowing [several persistent connections per domain] adds more queues, so one blocked queue doesn't stop the rest. RFC2616 allowed a client 2; browsers now allow more, 6 in Chrome.
  > This doesn't fix HTTP itself; it just spreads the risk over more TCP connections, which then fight over bandwidth and can starve the requests that matter most.
  2. **Domain sharding**
  If one domain gets 6 connections, use more domains: spread things over several subdomains of `google.com` that point at the same server, get more parallel connections, and ease head-of-line blocking further.

---

# How HTTP2.0 improves on HTTP1.x (performance)
2.0 is about performance; security is HTTPS's job, and it already does it well

## 1. Header compression

HTTP/1.1 could compress bodies (`Content-Encoding`), but not the headers themselves.
HTTP1.x headers are large and resent with every request (a GET is almost all headers), so HTTP/2 compresses them with **HPACK**.

HPACK, built for HTTP/2, does two main things:
- Both sides keep a [hash table] of header fields, so a value seen before is sent as a short index (0, 1, 2, ...) instead of in full, which trims and reuses headers a great deal.
<img src="/technology/network/http/compressheader.png" alt="compressheader.png" width="70%">

    > Tip
    HTTP/2 drops the start line: the method, URI and status code become header fields with a ":" prefix to set them apart.
- It also applies [Huffman coding to integers and strings], giving the most frequent characters the shortest codes for a high compression rate.

## 2. Multiplexing
HTTP1.1 only works around head-of-line blocking with more TCP connections; HTTP2.0 solves it inside the HTTP protocol itself.

**The new binary format (Binary Format) [binary framing]**

  HTTP1.x messages (mainly the headers) are **text**. (Text can stand for anything: text, images, video and so on)

  Text is ambiguous for machines (is a line break content or a separator?) and needs a state machine to parse, which is slow, so HTTP/2 switches every message to binary.

  A message is split into binary **frames**: `Headers` frames for the header fields and `Data` frames for the body. Frames don't have to arrive in any order, so nothing waits in line behind anything else, and head-of-line blocking at the HTTP level is gone.

A **bidirectional sequence of binary frames** between the two sides is a **stream (Stream)**. HTTP/2 runs many streams over one TCP connection, which is what [multiplexing] means.

> **If frames arrive out of order, how are they put back together?**
Only frames of different streams interleave; frames with the same Stream ID always arrive in order, and the receiver reassembles each stream into its request or response.

## 3. Setting request priority
An HTTP/2 frame looks like this:
<img src="/technology/network/http/binarystream.png" alt="binarystream.png" width="65%">
A frame has a **frame header** and a **frame body**; the header starts with a three-byte length of the body.

Next comes the **frame type**, roughly either a data frame or a control frame.
Data frames carry HTTP messages; control frames manage streams. [(For example, using a PRIORITY frame to change a stream's priority.)]

Then one byte of **frame flags** (8 bits), such as `END_HEADERS` (end of the header data) and `END_STREAM` (this direction is done sending).

The last 4 bytes are the `Stream ID`, the **stream identifier** the receiver uses to pick out and reassemble each stream's frames.

A normal request and response goes like this:
<img src="/technology/network/http/request&amp;response.png" alt="request&amp;response.png" width="50%">
Both sides start idle; the client's Headers frame opens a stream with a new `Stream ID`, the server opens its side when it receives it, and then both can send data and control frames.

To close, the client sends `END_STREAM` and becomes half-closed: it can still receive but not send.

The server, on receiving `END_STREAM`, is half-closed the other way round (it can send, not receive), then sends its own `END_STREAM`, and the stream is closed.

Each new stream takes the next ID; when the IDs run out, a new TCP connection starts counting again. The field is 4 bytes with the top bit reserved, so there are 2 to the 31st power IDs, about 2.1 billion.

Stream characteristics:
  - Concurrency. One HTTP/2 connection carries many frames at once, unlike HTTP/1; this is the basis of multiplexing.
  - Incrementing. Stream IDs are never reused; they only go up, and a new TCP connection starts over.
  - Bidirectionality. Client and server can both open streams, and either can send or receive.
  - Settable priority. Frames can be prioritized so the server handles important resources first.


## 4. Server Push
An HTTP/2 server doesn't just answer requests; it can open streams itself. When the browser asks for an HTML file, the server can push the resources that HTML references along with it, so the client waits less.

---

# HTTPS

HTTPS isn't a new application-layer protocol; it's HTTP with its communication layer swapped for SSL/TLS.

SSL, the **Secure Sockets Layer**, sits at the session layer (layer 5) of the OSI seven-layer model.
After three major versions, SSL was standardized as **TLS (Transport Layer Security)**; TLS/1.2 is the mainstream version now.

HTTPS adds three things over HTTP:
- Authentication: a third party can't impersonate the server (or the client)
- Privacy: content is symmetrically encrypted with a unique key for each connection
- Integrity: content is checked for tampering in transit

**In short, HTTPS is HTTP wrapped in SSL.**
With SSL, HTTP gains encryption, certificates and integrity protection.

<img src="/technology/network/http/https.png" alt="https.png" width="50%">

TLS/SSL rests on three kinds of basic algorithms:
- Asymmetric encryption: for authentication and key exchange
- Symmetric encryption: for encrypting data with the agreed key
- Hash functions: for checking integrity

## Solving possible eavesdropping on content: encryption
Asymmetric encryption is used to exchange a key, then symmetric encryption for the rest of the conversation.

In practice:
The sender encrypts a symmetric key with the receiver's public key, the receiver decrypts it with its private key, and from then on both sides use that symmetric key. So HTTPS combines both kinds of encryption.

## Solving possible tampering with messages: digital signatures
Data passing through many intermediate nodes can't be read, but it could still be altered; a digital signature is how integrity gets checked.

A digital signature does two things:
- It proves the message came from the sender, since nobody else can produce the sender's signature.
- It proves the message wasn't changed along the way.

    **How a digital signature is generated:**
    <img src="/technology/network/http/generatesignature.png" alt="generatesignature.png" width="85%">

    Sender: **hash the text into a digest**, **encrypt the digest with the sender's private key** to get the signature, and send it along with the text.

    **How a digital signature is checked:**
    <img src="/technology/network/http/checksignature.png" alt="checksignature.png" width="85%">

    Receiver: **decrypt the signature with the sender's public key** to get the digest back, **hash the received text** again, and compare the two; a match means the message arrived intact.

## Solving possible impersonation of the communicating parties: digital certificates

Certificate Authority (CA for short)
A CA is a third party that both client and server trust.

How a CA issues a certificate:
- The server's operator sends the CA its public key, organization info and domain, and applies for certification;
- The CA checks the details online and offline (does the organization exist, is the business legitimate, does it own the domain, and so on);
- If everything checks out, the CA issues a **certificate**.
A certificate holds, in plaintext, **the applicant's public key**, the applicant's organization details, **information about the issuing CA**, the validity period and the serial number, plus a **signature**.
The signature is made by **hashing** that plaintext into a **digest** and encrypting the digest with the CA's private key;
- When the Client connects, the Server sends its certificate;
- The Client hashes the certificate's plaintext the same way, decrypts the signature with the CA's public key, and compares the result with the **digest**; a match proves the certificate, and so the server's public key, can be trusted.
(Trusted CAs' public keys come built into the browser.)
- The client also checks the certificate's domain, validity period and so on; if the issuing CA isn't one it trusts, it can't find that CA's certificate and rejects the server's certificate.
<img src="/technology/network/http/certificateauthority.png" alt="certificateauthority.png" width="75%">


Reference: [An in-depth look at how HTTPS works](https://github.com/ljianshu/Blog/issues/50) (in Chinese)

Finally:

![httpsflow.png](/technology/network/http/httpsflow.png)


# Summary
- HTTPS optimizes HTTP for security
- HTTP2.0 optimizes HTTP for performance

<img src="/technology/network/http/summary.png" alt="summary.png" width="65%">
