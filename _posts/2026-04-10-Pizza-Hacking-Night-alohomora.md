---
layout: post
title: ZHAW Pizza Hacking Night
date: 2026-04-10
description: On-site event report and write-ups of solved challenges.
tags: writeups on-site web forensics cryptography
categories: writeups web forensics cryptography
author: "Anonymous Student"
---

## Pizza Hacking Night Experience

ZHAW organised an on-site event about CTFs, with, as the name suggests, free pizza, an expert talk by the events sponsor Orange Cyberdefense, and some CTF challenges by the Swiss Hacking Challenge (SHC) team /mnt/ain.

The evening began with interesting conversations with like-minded students and delicious pizza. There was a nice atmosphere, and you could tell that people were really into cybersecurity.

The first talk of the evening, "It's just a bug... Until it's Domain Admin!", was given by Fabrice Caralinda (Deputy Head of the Offensive Security Team at Orange Cyberdefense). In it, he spoke about a penetration test his team had carried out for a client as part of their day-to-day work. Overall, the talk was very engaging. I was both impressed and somewhat alarmed by the mistakes even large companies make in reality, the vulnerabilities that often exist in their systems, and how these could be - and indeed are - exploited by hackers.

After a short break, Marc Bollhalder from the Swiss Hacking Challenge gave an introduction to CTFs. He explained what they are and how they have evolved over the years. He rightly pointed out that AI is increasingly becoming a "problem", as AI systems can provide significant assistance in solving CTFs - and in some cases can even solve challenges entirely on their own - meaning that human skill is becoming less and less important. He also mentioned that the teams taking part in CTF events are becoming increasingly professional and that the difficulty of the challenges has, in some cases, reached absurd levels. Nevertheless, he believes that CTFs remain a great way to engage with the topic of cybersecurity and improve one's own skills. He then went on to present the Swiss Hacking Challenge and outlined the opportunities and offerings available to those interested.

Overall, I really enjoyed both talks. It was great to see what people who deal with hacking in their day-to-day work actually do. It was also interesting to hear about the experiences of someone who has been an active member of the CTF community for several years.

Following these talks, the Swiss Hacking Challenge organised a number of CTF challenges for us to tackle.

## Write-Ups

Out of the CTF challenges provided, I managed to solve a total of three. Below are brief write-ups explaining how I solved these challenges.

---

- **Challenge Name:** mc-ronald
- **Category:** Web
- **Description:** McRonalds, Switzerland's most questionable fast food chain, just launched online ordering. Their menu looks normal enough, but there is one item that seems... different.
- **Provided Files / URL:**
  - Website running the webshop
  - Source files of the webshop

{% include figure.liquid loading="eager" path="assets/img/posts/2026-04-10-pizza-hacking-night-alohomora/mc-ronalds.png"
class="img-fluid rounded z-depth-1" max_width="500px"%}

On the webshop you first had to create an account, after which you received a credit of CHF 5.00. The shop had five products on offer, with product 5 being called "The Secret Recipe". This product was available at a price of CHF 99999.00.

I then had a look at the source files of the webshop. After studying the `app.py` I found out the following:

- The product which leads to the flag has ID 5, i.e. the one with the high price tag
- When you buy a product, the frontend sends the product ID and the price to the backend and the backend uses those values directly
- When you buy a product, the backend checks if the price is not negative and if the user has enough money to buy the product
- The flag is presented on the "purchases" site, if the user managed to buy the product with ID 5

We don't have enough money to buy the product with ID 5 directly, but since the backend uses the values sent by the frontend, we may be able to change the price of product 5 to be less than the amount of money we have.

The first thing I tried was to change the price in the purchase form on the website. Unfortunately, the webshop checks if the form is tampered with and doesn't allow the values to be changed. I then just tried to order the product, which of course returned an error message stating that I didn't have enough money, but I now had a valid request. I copied the request, changed the price of the product directly in the request body and then sent this new request to the backend. After that I had a look at the "purchases" site where the flag was displayed: `ZHAW{...}`

---

- **Challenge Name:** haj-in-the-middle
- **Category:** Forensics
- **Description:** FTPS? what's that, we just use good old FTP here, no need for all that encryption overhead.
- **Provided Files / URL:** `dump.pcapng`

{% include figure.liquid loading="eager" path="assets/img/posts/2026-04-10-pizza-hacking-night-alohomora/wireshark.png"
class="img-fluid rounded z-depth-1" max_width="500px"%}

The captured network traffic included 57 packets, whereby most of them were FTP packets. Like the description of the challenge already hints at they were not encrypted.

After looking through all the packets I found out the following:

- It seems like a user logged in to a server, did some actions on the server and finally uploaded a file
- The user used the username `sebi` and password `dadada` to log in to the server
- The user uploaded a file called `Secrets.kdbx`
- Packet number 51 is the only FTP-DATA packet, which is also the largest packet with a length of 2135

I then thought that the `Secrets.kdbx` file probably contains the flag, so I exported the FTP data from packet number 51 into a file called `Secrets.kdbx`. Since a `.kdbx` file is a KeePass database file, I downloaded and installed KeePass on my computer. Then, I tried to open the file with KeePass, where I was asked to enter the master key. Since the only password I had available was `dadada` which the user `sebi` used to log in to the server, I tried that one, which directly worked.

{% include figure.liquid loading="eager" path="assets/img/posts/2026-04-10-pizza-hacking-night-alohomora/keepass.png"
class="img-fluid rounded z-depth-1" max_width="500px"%}

The database contained only one entry called `flag`, and the password field contained the actual flag: `SCD{...}`

---

- **Challenge Name:** really-secure-application
- **Category:** Cryptography
- **Description:** We recently had an intern around before the crisis happened. He learned about cryptography and encrypted the password for our coffee machine??? We can't get through this without coffee.
- **Provided Files / URL:**
  - `main.py`
  - `output.txt`

The file `main.py` contained the following code, which was probably used to encrypt the flag with RSA:

```python
from Crypto.Util.number import getPrime, inverse, bytes_to_long, long_to_bytes
import random

flag = b"SCD{f4k3_fl4g}"

p = getPrime(1024)
q = 7

n = p * q

e = e = 65537

phi = (p - 1) * (q - 1)
d = inverse(e, phi)

flag_int = bytes_to_long(flag)
ciphertext = pow(flag_int, e, n)

print(f"Public modulus (n): {n}")
print(f"Public exponent (e): {e}")
print(f"Encrypted flag: {ciphertext}")

decrypt = pow(ciphertext, d, n)
# print(long_to_bytes(decrypt))
```

The file `output.txt` contained the values of the public modulus (n), the public exponent (e) and the encrypted flag.

To decrypt the ciphertext we need the values `d` and `n`. `n` is given, but `d` we still need to compute, which is the inverse of `e` mod `phi`. `e` is also given, but we don't know `phi` yet. To calculate `phi` we need `p` (unknown) and `q` (given). Since we know `n`, which was calculated by multiplying `p` with `q`, we can calculate `p`, which means that we have everything we need to decrypt the flag.

Doing all the calculations just mentioned in reverse order finally revealed the flag: `SCD{...}`

## Conclusion

All in all, the Pizza Hacking Night was a great event. It was nice to meet like-minded people, listen to interesting talks, and play CTFs with others in person for a change.
