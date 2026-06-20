---
layout: post
title: Pizza Hacking Night Spring 2026 - On-site impressions and writeup of Treasure Hunt 1
date: 2026-06-22
description: Extracting a flag from ZIP metadata using compression ratios instead of breaking encryption
tags: writeups rev on-site pizzahackingnight
categories: writeups rev
author: "Guillaume"
---


The Pizza Hacking Night was my second on-site CTF, it was held at a ZHAW building in Winterthur. It started with free pizza distribution then there was a presentation by Fabrice Caralinda for Orange Cyberdefence, the main sponsor of the event, about a real case of pentestinf he did with his team. I liked this presentation because it showed me that some skills needed to do CTFs can also be useful in real life. This presentation was followed by another one by the Swiss Hacking Challenge team /mnt/ain, which created the challenges for this event, about CTFs in general and particularly the competitive/professional approch of CTFs. I found it interessting because I wasn't aware that some people spend so much time preparing CTFs.

I really enjoyed the athmosphere of the event and even if I only managed to solve simple challenges I got a 4th place out of 20, I wasn't excpecting such a high result so that was a good surprise.

Here is the writeup of one of the challenges I solved during the night.

## Treasure Hunt 1
- **Name of the CTF Event:** Pizza Hacking Night - Spring 2026
- **Challenge Name:** Treasure Hunt 1
- **Category:** rev
- **Provided Files / URL:** `door (a binary file)`
- **Goal:** Find the flag.

## Initial Analysis

By using Ghidra I was able to deternime that the user can input a phrase that is safely retrived by this instruction : `fgets(phrase,0x200,stdin)`.\
This phrase is later compared to a `correct_phrase` using `iVar2 = strcmp(phrase,correct_phrase);` leading to a print of the flag later if the phrase is the same as the correct phrase.

## Getting the correct phrase

However the correct phrase is not stored in plain text in the binary but is the result of a series of operations on an `encrypted_phrase` :
```C
i = 0;
while( true ) {
  phrase_len = strlen(encoded_phrase);
  if (phrase_len <= (ulong)(long)i) break;
  correct_phrase[i] = encoded_phrase[i] ^ 0x42;
  i = i + 1;
}
```

Since the operations are simply some XORs a small Python program can find the correct phrase :
```py
encrypted_flag = "-2',b-0b+..b%'6b/;b,7)'1"  # Taken directly from the binary
print(''.join(map(lambda x: chr(ord(x) ^ 0x42), encrypted_flag)))
```
```
open or ill get my nukes
```

Now that I know the correct phrase I can just pass it to the program to retrive the flag :
```
After your first adventure you have reached yet another mysterious door. You try to open it with force but you fail miserably. After some searching you find some ancient symbols on the wall telling you to whisper!? a secret phrase which might open the door...
open or ill get my nukes
The door opened and you have sucessfully reached the next room realing another secret:
ZHAW{...}
```
