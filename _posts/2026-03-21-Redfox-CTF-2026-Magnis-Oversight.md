---
layout: post
title: Redfox CTF 2026 - Magni's Oversight (Forensics)
date: 2026-03-21
description: How the "Magni's Oversight" challenge was solved at the Redfox CTF 2026 event.
tags: writeups forensics redfoxctf
categories: writeups forensics
author: "Anonymous Student"
---

## Challenge Overview

- **Name of the CTF Event:** Redfox CTF 2026
- **Challenge Name:** Magni's Oversight
- **Category:** Forensics
- **Description:** Magni might be the God of Force, God of Brutality, God of Might, God of Physical Strength, God of Survival, God of Brotherhood, God of Family, God of Loyalty and the Son of Thor. But he made an oversight and someone sneaked into Asgard. Can you spot what he couldn't?
- **Provided Files / URL:** `file.pcap`
- **Goal:** Find the flag.

## Initial Analysis

{% include figure.liquid loading="eager" path="assets/img/posts/2026-03-21-magnis-oversight/wireshark-1.png"
class="img-fluid rounded z-depth-1" max_width="500px"%}

After opening the provided `.pcap` file in Wireshark, I saw that it contained a lot of different packets, so I used Wireshark's statistics tools to get a first overview. The recorded network traffic contained...

- A total of 370 packets
- 33 UDP packets
  - 3 NetBIOS Name Service
  - 30 MDNS
- 332 TCP packets
  - 6 TLS
  - 1 SSH
  - 1 HTTP
  - 248 FTP Data
  - 1 FTP
- 5 ARP packets

By looking at these numbers I came up with the following hypotheses:

1. There are six TLS packets, i.e. six encrypted messages. Maybe these messages were encrypted using a simple encryption scheme, or they were all encrypted with the same key, so it may be possible to decrypt them and one of the messages may contain the flag.
2. There is only one SSH packet, one HTTP packet and one FTP packet. Either one of those packets contains the flag directly, or they probably contain some other useful information.
3. There are 248 FTP data packets. Maybe their payloads have to be combined somehow to form a single file, which might then contain the flag or other files with useful information.

## Solution Path

First, I had a look at the single SSH, HTTP and FTP packets. The SSH packet had the following content:

```
SSH request - File: T09wcyBTb3JyeS4uLiBUaGlzIGlzIGEgcmFuZG9tIHRleHQK
```

The file content wasn't directly readable, so I tried to decode it from different formats, whereby decoding it from Base64 finally revealed the following:

```
OOps Sorry... This is a random text
```

I then continued with the HTTP packet:

```
GET /download?file===gC9NjMxM3clN2YB9FZlpXay9Ga0VXYuV1eY9kREVkU HTTP/1.1
Host: example.com
```

Here, the file parameter also looked like being encoded, similar to the content of the SSH packet from before, so I also tried to decode it. Unfortunately this didn't lead to anything readable.

I then continued with the FTP packet, which had the following content:

```
FTP request - File: T2gsIHRoaXMgaXNuJ3QgeW91ciBmbGFnLCA=
```

Like the previous messages, the file content looked like being encoded, and here I also noticed the `=` at the end, which is typical for Base64, so I tried decoding it from Base64 which revealed the following:

```
Oh, this isn't your flag,
```

Since none of these three messages contained anything useful, I thought that they were probably just decoy packets, and that I should focus on one of my other hypotheses instead. Just before moving on, I recognized a TCP packet, which was sent directly after the previously investigated packets, which looked different from the other TCP packets in the traffic.

{% include figure.liquid loading="eager" path="assets/img/posts/2026-03-21-magnis-oversight/wireshark-2.png"
class="img-fluid rounded z-depth-1" max_width="500px"%}

In the payload of this TCP packet I found the following POST request:

```
POST /upload HTTP/1.1
Host: example.com

File: VHJ5IGhhcmRlcg==
```

Decoding the file content from Base64 revealed the following message:

```
Try harder
```

I then turned my attention to the six TLS packets. Four of them were at the start of the recorded traffic, one after the other, and the remaining two packets were near the end of the traffic. I examined all six packets and extracted the encrypted application data they contained in hexadecimal form:

```
1) bb1d5021e59b06e1ed7ccc42096deac2fb4d789f17b87d034d3dfca536fdb447619a
2) c3c3025e23dc555f3fef582c67e0ddfb1e23a158b3c5e87cf609ec09db0cfd35d99f
3) 8e7155bbf1739f01bad73b4fa3e8636ec7201f2da99defc61940437f2661e2c37b09
4) 5b5a2edad2fa5b237ec84f344537c101813dd94db3ec5e0a0680f638ab2f3f2c10c5
5) b54b301b713ac56dd8729f35165742476ba9440ba6d80b49c0c3ddd6f3474b815d1e
6) d18d555f7e63733e0fdd45fe07edf6b326a05b6f4d019aa9ddf8c4971a3acb0fdc30
```

The easiest way to decode the messages would be to use the matching key, but until now I didn't have one. Since the previously investigated packets didn't contain a key, I looked through all remaining packets in the traffic, except the FTP data packets, hoping to find something that could be used as a key. Unfortunately there was nothing that looked promising.

I then thought back to a past CTF challenge, where I also had multiple encrypted messages without the matching key. There I used a method called "Crib Dragging" to decrypt the messages. I opened the website [Crib Drag Attack Tool](https://gfuscht.cedricgasser.com/cribdrag), which I already used for that other challenge, and tried to decrypt the messages. After trying the known flag format `REDFOX{...}` and multiple other, often used words on all six messages, I didn't get anything that looked promising.

I also tried multiple "simple" encryption/decryption schemes like ROT13, ROT47 or XOR with some simple keys, but nothing seemed to work.

Moving on to my last hypothesis, I investigated the many FTP data packets. All packets had a payload size between 120 and 123 bytes. Interestingly, the protocol used wasn't the same for all packets. Some were smtp, some smb, some mdns and some telnet. Here are some payload examples of these packets:

```
smtp request - Data: WORLm9F2M9P16bw90WMs08fLrDtZhgOildHzcUZU5Xu5QP7vvMNdNkCw1tY6RY7cWNRMFdOTepvPrwAejrH7s0NBmjTkWLsSZt8F
smb request - Data: VkboWz7wLMxXqn0UELaBmZkfdRQLOV7AivKj6HEUm3VPGk910tt6AOpUUvvcMIe0R0JSsqbGy64jhECDN87k6s6gaj9gVoD8VDwA
mdns request - Data: i0bdRR4g75n1tRcyRTSXRK88s2Tpd7I0IhY2ZAXoEURkm0QU7tW7EWL7RsBJx6CN9XNzEzPUxm2gMGjX8U6EAU9hm9wriDIm0io6
telnet request - Data: 9xQotsWgeLV6lMtCAI16OYHuFxL7F67xRrLbfO9zAbGLZzmVjw5ixvQj9ru3cyA4G2Vl5Ylt16ywsfGOePGym5tRZVaddsANvEJs
```

Decoding individual messages didn't yield anything readable, so I also tried concatenating several consecutive messages, as well as only messages that used the same protocol, but nothing worked. Trying to interpret the hex values at the beginning of messages as file signatures also didn't lead to anything useful.

Since nothing has worked so far, I thought I might have overlooked a small detail that could be the key to success, so I went back to the very first packet and took another careful look at all the packets. Once I reached the single HTTP packet, something struck my attention.

```
GET /download?file===gC9NjMxM3clN2YB9FZlpXay9Ga0VXYuV1eY9kREVkU HTTP/1.1
Host: example.com
```

The `===` in the URL seemed strange. Normally there is only one `=`, i.e. `file=...`. Perhaps the second and third `=` are part of the parameter value. I tried decoding `==gC9NjMxM3clN2YB9FZlpXay9Ga0VXYuV1eY9kREVkU` from Base64, but the output wasn't something readable. Then I thought that `=` usually appears at the end of a Base64 string and not at the beginning, so perhaps the value needs to be read backwards. Therefore, I reversed the value and tried to decode it from Base64 again, which finally revealed the flag:

```
REDFOX{...}
```

## Conclusion

All in all, this was a pretty interesting challenge. Compared to my last forensics CTF challenge, which was also a network traffic analysis, there were now much more packets in this challenge and way less hints. That made it much harder to figure out where to start and what to try. But looking back, I think that the initial analysis using Wireshark's statistics tools and trying to come up with different hypotheses up front, were good ideas. That gave me a sort of roadmap of things I could try, rather than just trying different things without a clear goal.

In the end, many things I tried didn't work out, and the key to solving this problem lay, once again, in a small, easily overlooked detail and a bit of creativity. The title of the challenge, "Magni's Oversight", turned out to be spot on...
