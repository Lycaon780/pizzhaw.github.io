---
layout: post
title: Pizza Hacking Night Spring 2026 - On-site Impressions and Challenge Writeup by Nevos
date: 2026-04-11
description: Extracting a flag from ZIP metadata using compression ratios instead of breaking encryption
tags: writeups misc forensics steganography pizzahackingnight
categories: writeups misc
author: "Nevos"
---

This was my first on-site CTF event, held at the Technikum in Winterthur. The main sponsor was Orange Cyberdefense, who gave a talk about their work. The Team /mnt/ain from the Swiss Hacking Challenge was also present; they spoke about their group and CTF events in general, and they were responsible for creating the challenges for this event. As the name suggests, there was free pizza and plenty of goodies for all participants.

The presentation from Orange was very interesting and provided great insight into their operations. It particularly highlighted how similar CTF events are to the "real thing." The SHC presentation focused on the broader world of CTFs, offering excellent advice, tips, and a general overview of the competitive landscape.

Beyond the informative presentations, the atmosphere of an on-site event was quite different from the online ones I have done previously. While online events often feel more "laid-back," being physically present with other participants created a much more competitive feel. It was also great to be able to speak directly with others in person rather than relying solely on online chats.

Though perhaps not the most strategic approach for maximizing points, I immediately dived into the more difficult challenges instead of starting with the easier ones. In the end, I managed to solve two of those before I had to leave. I was stuck on one for a significant amount of time but eventually solved it, which I was very happy about—especially since I finished it relatively quickly compared to my usual pace during online challenges.

Overall, it was an educational and enjoyable evening. I’m definitely looking forward to attending another on-site CTF event soon to experience this again.

Two images I took, first one was during the event, and the second one as I left:

{% include figure.liquid loading="eager" path="assets/img/posts/2026-04-11-compress-me-baby/onsite_1.jpg"
class="img-fluid rounded z-depth-1" max_width="500px" zoomable=true%}

{% include figure.liquid loading="eager" path="assets/img/posts/2026-04-11-compress-me-baby/onsite_2.jpg"
class="img-fluid rounded z-depth-1" max_width="500px" zoomable=true%}

The rest of the writeup is about one of the two challenges I managed to solve there.

## Compress Me Baby One More Time - Challenge Overview

- **CTF Event:** Pizza Hacking Night - Spring 2026 Edition
- **Challenge Name:** Compress Me Baby One More Time
- **Category:** Misc
- **Description:** _*zips you*_
- **Provided Files:** `compress-me-baby-one-more-time_tar.gz`
- **Flag Format:** `ZHAW{...}`

## Goal

Figure out what the compressed archive is hiding and extract the flag.

## Initial Analysis

The challenge name and the description both heavily hint at compression. The name itself, "compress me baby one more time", indicates something being compressed multiple times.The download is a `.tar.gz` archive. Extracting it shows a single zip file called `quantum_encryption_2026.zip`:

```bash
tar xzf compress-me-baby-one-more-time_tar.gz
```

Extracting that zip in turn produces **25 smaller zip files**, numbered `000.zip` through `024.zip`:

```bash
unzip quantum_encryption_2026.zip
```

Each one has a single `.txt` file with seemingly random three-word names built from a small wordlist — things like `crazy_hacker_digital.txt`, `uwu_doggo_once.txt`, `pizza_kitty_crazy.txt` and so on.

### Tools Used

- `tar`, `unzip`, `zipinfo`, `fcrackzip`, `zip2john`
- Python 3 (`zipfile` module)

# Solution Path

## Step 1: Attempting Extraction

Trying to extract any of the nested zips immediately fails:

```bash
$ unzip 000.zip
   skipping: crazy_hacker_digital.txt  unable to get password
```

They're all password-protected. My first thought was to try cracking them.

### Dead End: Password Cracking

I spent a lot of time trying to brute-force the passwords. I used `fcrackzip` with both dictionary and brute-force modes. It reported several "possible" passwords for each zip, but every single one of them produced `invalid compressed data to inflate` errors when actually used to extract.

I also tried the filenames as passwords, words from the filenames, challenge-related strings like `quantum`, `compress`, `baby` — nothing worked.

## Step 2: Examining the Metadata

After a while, I decided to use a different approach. I used `zipinfo` and Python's `zipfile` module to inspect the metadata of all 25 zips more carefully:

```bash
$ zipinfo 000.zip
-rw-rw-r--  3.0 unx  3600 TX  40 defX 26-Apr-08 23:03 crazy_hacker_digital.txt
```

Two things immediately stood out. First, the compression ratios were extreme, so the plaintext files are almost certainly only something like a single character repeated many times. Second, the `uncompressed_size` and `compressed_size` values varied across all 25 files in a way that felt a bit weird or unusual. Bascially, it seemed like something that was done deliberately, rather than random.

I dumped the sizes for all files:

```python
import zipfile

for i in range(25):
    z = zipfile.ZipFile(f'{i:03d}.zip')
    info = z.infolist()[0]
    print(f'{i:03d}.zip: size={info.file_size}, compress_size={info.compress_size}')
```

```
000.zip: size=3600, compress_size=40
001.zip: size=2815, compress_size=39
002.zip: size=2470, compress_size=38
003.zip: size=3570, compress_size=41
004.zip: size=5290, compress_size=43
...
```

## Step 3: The Ratio of the Files is the Clue

Looking at the first file: `3600 / 40 = 90`. And 90 in ASCII is `Z`. The second one: `2815 / 39 = 72.18`, rounded down to `72`, which is `H`. Third one: `2470 / 38 = 65`, which is `A`.

`Z`, `H`, `A`... this is spelling out the flag format. Integer division of `file_size` by `compress_size` gives the ASCII code for each character:

```python
import zipfile

flag = ""
for i in range(25):
    z = zipfile.ZipFile(f'{i:03d}.zip')
    info = z.infolist()[0]
    char_code = info.file_size // info.compress_size
    print("Zip: " + f'{i:03d}.zip' + " | Uncompressed: " + str(info.file_size) + " | Compressed: " + str(info.compress_size) + " | Ratio (floor): " + f'{char_code:3d}' + " | Char: " + chr(char_code) + " |")
    flag += chr(char_code)

print()
print("Full flag: ", flag)
```

{% include figure.liquid loading="eager" path="assets/img/posts/2026-04-11-compress-me-baby/finding_flag.png"
class="img-fluid rounded z-depth-1" max_width="500px" zoomable=true%}

## Flag

```
ZHAW{wh4t_1n_th3_r4t10??}
```

## Conclusion

The password-protected zips were a complete red herring, the flag was never inside the files. Instead, the challenge author deliberately created the uncompressed and compressed sizes of each zip so that their integer division would encode an ASCII character.

The key takeaway here is that **metadata is data too**. When the obvious path (cracking passwords) goes nowhere, it's worth looking at what information is available without ever opening the files. Maybe this might even be the obvious first step, instead of immediately jumping to attempting to crack the passowrds, which can take a lot of time. In this case the answer was literally in the file sizes.
