---
layout: post
title: Sekai CTF - 6-7 Puzzle Quiz
date: 2026-06-29
description: Solving problems leads to a secret one
tags: writeups misc
categories: writeups misc
author: "Guillaume"
---

## Challenge Overview

- **Name of the CTF Event:** Sekai CTF
- **Challenge Name:** 6-7 Puzzle Hunt
- **Category:** 67
- **Description:** Six (or seven?) handcrafted logic puzzles. Best enjoyed with a pencil, not a solver.
- **Provided Files / URL:** https://67-hunt.chals.sekai.team/
- **Goal:** Solve the seventh puzzle

## Initial Analysis

The website contains 6 different logical puzzles but the description is hinting that a secret seventh puzzle might exist.

# Solving the puzzles

I started by using a solver to solve the puzzles because they would usualy require a lot of time and backtracking to manually find a solution.\
Here are the solved puzzles:

## Tapa
![tapa](../assets/img/posts/2026-06-29-Sekai-CTF-6-7-Puzzle-Hunt/tapa.png)

## Nurikabe
![nurikabe](../assets/img/posts/2026-06-29-Sekai-CTF-6-7-Puzzle-Hunt/nurikabe.png)

## Fillomino
![fillomino](../assets/img/posts/2026-06-29-Sekai-CTF-6-7-Puzzle-Hunt/fillomino.png)

## Shikaku
![shikaku](../assets/img/posts/2026-06-29-Sekai-CTF-6-7-Puzzle-Hunt/shikaku.png)

## Scyscrapers
![scyscrapers](../assets/img/posts/2026-06-29-Sekai-CTF-6-7-Puzzle-Hunt/scyscrapers.png)

## Kakuro
I actually solved that one manually (not on the first try I must admit) because it looked fun and faster to solve than the other ones.
![kakuro](../assets/img/posts/2026-06-29-Sekai-CTF-6-7-Puzzle-Hunt/kakuro.png)


# The last puzzle
All of the 6 puzzles gave a Tetris like piece with letters in it.\
I look at the source code of the webpage to see if anything related to a seventh puzzle appeared but I didn't found anything relevant.\
I then looked for a solver that would solve a problem with such pieces but I didn't found anything neither so I tried to solve it manually with nothing but a text editor.\
Since the flag is formated as `SEKAI{...}` I started by placing the first pieces in the following way :
```
┏━━━━━━━━━━━━━━━┓     ┏━━━━━━━┓
┃ S   E   K   A ┃     ┃ {   S ┃
┗━━━━━━━┓   ┏━━━┛     ┗━━━┓   ┃
        ┃ I ┃             ┃ E ┃
    ┏━━━┛   ┃             ┃   ┃
    ┃ 6   7 ┃             ┃ K ┃
    ┗━━━━━━━┛             ┃   ┃
                          ┃ A ┃
                          ┃   ┃
                          ┃ I ┃
                          ┃   ┃
                          ┃ } ┃
                          ┗━━━┛
```
Then since only one piece can complete the `SEKAI` without overlapping I got :
```
┏━━━━━━━━━━━━━━━┓━━━┏━━━━━━━┓
┃ S   E   K   A ┃ I ┃ {   S ┃
┗━━━━━━━┓   ┏━━━┛   ┗━━━┓   ┃
        ┃ I ┃ B   B   L ┃ E ┃
    ┏━━━┛   ┃   ╺━━━┓   ┃   ┃
    ┃ 6   7 ┃ _   L ┃ I ┃ K ┃
    ┗━━━━━━━┗━━━━━━━┻━━━┃   ┃
                        ┃ A ┃
                        ┃   ┃
                        ┃ I ┃
                        ┃   ┃
                        ┃ } ┃
                        ┗━━━┛
```
After this I noticed that the G-shaped piece would perfectly fit at the right :
```
┏━━━━━━━━━━━━━━━┓━━━┏━━━━━━━┓
┃ S   E   K   A ┃ I ┃ {   S ┃
┗━━━━━━━┓   ┏━━━┛   ┗━━━┓   ┃
┃ C   R ┃ I ┃ B   B   L ┃ E ┃
┃   ┏━━━┛   ┃   ╺━━━┓   ┃   ┃
┃ _ ┃ 6   7 ┃ _   L ┃ I ┃ K ┃
┃   ┗━━━━━━━┗━━━━━━━┻━━━┃   ┃
┃ E ┃ _ ┃               ┃ A ┃
┃   ╹   ┃               ┃   ┃
┃ _   S ┃               ┃ I ┃
┗━━━━━━━┛               ┃   ┃
                        ┃ } ┃
                        ┗━━━┛
```
I then combined the two remaining pieces to get :
```
        ┏━━━━━━━━━━━┏━━━┓
        ┃ I   T   S ┃ _ ┃
        ┃   ┏━━━┳━━━┫   ┃
        ┃ E ┃ X ┃ Y ┃ _ ┃
┏━━━━━━━┛   ┃   ╹   ╹   ┃
┃ N   T   E ┃ G   E   R ┃
┗━━━━━━━━━━━┛━━━━━━━━━━━┛
```
By putting everything together I got :
```
┏━━━━━━━━━━━━━━━┓━━━┏━━━━━━━┓
┃ S   E   K   A ┃ I ┃ {   S ┃
┗━━━━━━━┓   ┏━━━┛   ┗━━━┓   ┃
┃ C   R ┃ I ┃ B   B   L ┃ E ┃
┃   ┏━━━┛   ┃   ╺━━━┓   ┃   ┃
┃ _ ┃ 6   7 ┃ _   L ┃ I ┃ K ┃
┃   ┗━━━┏━━━━━━━━━━━┏━━━┓   ┃
┃ E ┃ _ ┃ I   T   S ┃ _ ┃ A ┃
┃   ╹   ┃   ┏━━━┳━━━┫   ┃   ┃
┃ _   S ┃ E ┃ X ┃ Y ┃ _ ┃ I ┃
┏━━━━━━━┛   ┃   ╹   ╹   ┃   ┃
┃ N   T   E ┃ G   E   R ┃ } ┃
┗━━━━━━━━━━━┛━━━━━━━━━━━┗━━━┛
```
And so by reading line-by-line I could retrive the flag.
