---
layout: post
title: V1t CTF 2026 - Quack CIA
date: 2026-06-29
description: OSINT leading to VIM undo file
tags: writeups osint vim
categories: writeups osint
author: "Guillaume"
---

## Challenge Overview

- **Name of the CTF Event:** V1t CTF 2026
- **Challenge Name:** Quack CIA
- **Category:** OSINT
- **Description:** I swear I saw flag.txt in the video, but now it’s redacted—IT MUST BE THE CIA. I SWEAR… BRO, THE CIAAAAAA!
- **Provided Files / URL:** https://www.youtube.com/watch?v=65wwer7yXLg
- **Goal:** \*Find the github repo in the video don't stalk me please 🥺

## Initial Analysis

The Youtube video shows someone hacking a (most likely) fake CIA website using a python script.

![yt-video.png](../assets/img/posts/2026-06-30-V1tCTF-Quack-CIA/yt-video.png)

## Fining the GitHub repository

Since the challenge goal mentions a public GitHub repository I logged in to GitHub to be able to search code in public repositories and looked for `V1t script kiddie tool 326532` since this is the first thing being printed by the scipt in the video.\
The first result was a script named `hack.py` from a repostory named `cia` so I check the repository and it had a file named `flag.txt`.

## Getting the flag

The `flag.txt` contained a base64-encoded string which can be decoded into `v1t{REDACTED}` I tried to submit the flag but of course it did not work. I then noticed that the repository had 2 commits and looked at the first one to see if the flag was different back then. The `flag.txt` file was the same but an additional binary file was present : `.flag.txt.un~`­. I downloaded it and ran the `cat` command on it :

```bash
$ cat flag.txt.un\~
VimUnDo��
J�M{m��lcNepP/0�O%̲ǋ<�iAk_�����iAU�5_�(����iAj�(djF0e3QwbW15X3NjcjFwdF9rMWRkMTNfMW4xdH0=5�%
```

The first line starting with `VimUnDo` I looked up what a `.un~` file is for Vim and found that it is a file storing the Vim editing history of a file. The `.un~` file only contains relative changes so I downloaded the associated `flag.txt` and opened it with Vim. I then trieand td the undo command but it didn't work and said `Already at oldest change`.I finally figured out that I had to use the command `:rundo flag.txt.un~`.\
After executing the command I was able to undo the last edit and the base64-encoded string changed to `djF0e3QwbW15X3NjcjFwdF9rMWRkMTNfMW4xdH0=` which I could decode to retrive the real flag.
