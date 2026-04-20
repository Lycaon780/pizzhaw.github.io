---
layout: post
title: WreckCTF 2026 - rev/pipeline
date: 2026-03-24
description: Inversion of various Python functions
tags: writeups python reverse hash
categories: writeups reverse
author: "Guillaume"
---

## Challenge Overview

- **Name of the CTF Event:** WreckCTF 2026
- **Challenge Name:** pipeline
- **Category:** rev
- **Description:** flag -> pipeline :)
- **Provided Files / URL:** `challenge.py`
- **Goal:** Find the flag.

## Initial Analysis

The first thing I noticed was multiples classes named `_0`, `_1` until `_B` containing a method called `f`. There is also a list name `_Q` composed of 32 instances of thoses classes. At the end of the file there is a `main` function askip the user for an input and then processing it with the following function :

```python
def _c(f):
    if not (f.startswith(_PR) and f.endswith(_SU)):
        return False
    if not all(0x20 <= b <= 0x7E for b in f):
        return False
    x = bytearray(f)
    for o in _Q:
        o.f(x)
    return bytes(x) == _T
```

The first `if` is to check is the input is of the form `wreck{...}` and the second one is to assert that the input only contains printable characters. Then comes the interestiong part where the input is modified by the `f` method of every element of `_Q` before beeing compared to `_T`. So to find the flag I just have to reverse the `f` funtion of all of the 12 classes and then recover the flag from `_T`.

## Trivial reverses

I noticed that the `f` methods of the classes `_0`, `_6` and `_7` are symetrical so I could reverse those with just a call to the `f` method, for exemple :

```python
class _0:
    def __init__(s, k): s.k = bytes(k)

    def f(s, x):
        k = s.k
        for i in range(len(x)):
            x[i] ^= k[i % len(k)]

    def rev(s, x):
        s.f(x)
```

## Basic operations

I then moved on to the classes `_1` and `_2` which are realy similar :

```python
class _1:
    def __init__(s, k): s.k = bytes(k)

    def f(s, x):
        k = s.k
        for i in range(len(x)):
            x[i] = (x[i] + k[i % len(k)]) & 0xFF


class _2:
    def __init__(s, k): s.k = bytes(k)

    def f(s, x):
        k = s.k
        for i in range(len(x)):
            x[i] = (x[i] - k[i % len(k)]) & 0xFF
```

Thoses clases just perform basic addition or substraction before doing `& 0xFF`. This operation allows to keep only the first 8 bits of the result and prevents an overflow in `x[i]` which should contain a single byte only. It is also interesting to note that this is equivalent to doing a modulo 256. So I inverted the additions by doing substractions and vice-versa. For class `_1` this gives :

```python
class _1:
    def __init__(s, k): s.k = bytes(k)

    def f(s, x):
        k = s.k
        for i in range(len(x)):
            x[i] = (x[i] + k[i % len(k)]) & 0xFF

    def rev(s, x):
        k = s.k
        for i in range(len(x)):
            x[i] = x[i] - k[i % len(k)] & 0xFF
```

## Bits manipulation

The next class on the list is `_3` and features a cool bit trick :

```python
class _3:
    def __init__(s, a): s.a = a & 7

    def f(s, x):
        a = s.a
        if not a:
            return
        for i in range(len(x)):
            b = x[i]
            x[i] = ((b << a) | (b >> (8 - a))) & 0xFF
```

First of all the `__init__` method enforce that `a` is between 0 and 7. Then the `((b << a) | (b >> (8 - a))) & 0xFF` operation is a way of doing a rotation of bits to the left. To reverse it I just need to do the rotation to the right :

```python
def rev(s, x):
    a = s.a
    if not a:
        return
    for i in range(len(x)):
        b = x[i]
        x[i] = ((b << (8 - a)) | (b >> a)) & 0xFF
```

## Position swapping

The following class is :

```python
class _4:
    def __init__(s, p): s.p = list(p)

    def f(s, x):
        n = bytearray(len(x))
        for i, j in enumerate(s.p):
            n[i] = x[j]
        x[:] = n
```

Here the position of the elements of `x` are swaped based on the values the `p` attribute so I jusy have to swap them the other way around to revert the operation :

```python
def rev(s, x):
    n = bytearray(len(x))
    for i, j in enumerate(s.p):
        n[j] = x[i]
    x[:] = n
```

The next one is a bit similar :

```python
class _5:
    def __init__(s, t): s.t = list(t)

    def f(s, x):
        t = s.t
        for i in range(len(x)):
            x[i] = t[x[i]]

    def rev(s, x):
        t = s.t
        for i in range(len(x)):
            x[i] = t.index(x[i])
```

## Another XOR

In class `_8` each element is XOR with the precedent, starting from the second until the last one.

```python
class _8:
    def f(s, x):
        for i in range(1, len(x)):
            x[i] ^= x[i - 1]
```

To reverse it I just have to perform the same operations but in reverse order :

```python
def rev(s, x):
    for i in range(len(x) - 1, 0, -1):
        x[i] ^= x[i - 1]
```

## More advanced operations

Although the next class looks simple, it isn't because a standard division won't work to reverse a multiplication under a modulo :

```python
class _9:
    def __init__(s, k):
        s.k = bytes(k)
        for b in s.k:
            assert b & 1

    def f(s, x):
        k = s.k
        for i in range(len(x)):
            x[i] = (x[i] * k[i % len(k)]) & 0xFF
```

However the `__init__` method ensure that every element of `k` is odd which in thoses conditions allow the multiplication to be bijective. This property can be quickly tested using the following code :

```python
def check(n):
    for i, j in enumerate(sorted(map(lambda x: (x * n) & 0xFF, range(256)))):
        if i != j:
            print('doublon :', i)
```

I then could build a function able to reverse a multiplication modulo 256 :

```python
def inv_mod_256(x, n):
    if x == 0:
        return 0
    for i in range(256):
        if (i * n) % 256 == x:
            return i
    return -1
```

The reverse method is now fairly easy to do :

```python
def rev(s, x):
    k = s.k
    for i in range(len(x)):
        x[i] = inv_mod_256(x[i], k[i % len(k)])
```

## Hashing time

The two remaining classes looks more difficult to reverse because they use a hashing function which is by definition not reversible :

```python
def _mh(a):
    def h(d): return hashlib.new(a, d).digest()[:3]
    return h


class _A:
    h = staticmethod(_mh("md5"))
    def __init__(s, i, e=None): s.i = i; s.e = bytes(e) if e else None

    def f(s, x):
        i = s.i
        c = bytes(x[i:i + 3])
        d = s.h(c)
        x[i:i + 3] = bytes(p ^ q for p, q in zip(c, d))


class _B(_A):
    h = staticmethod(_mh("sha256"))
```

Howerver since only three bytes are hashed each time it is possible to store all the possible results on a dictionary to reverse the operation :

```python
rev_A = {}
md5 = _mh('md5')
for i in range(256 ** 3):
    c = i.to_bytes(3)
    d = md5(c)
    key = bytes(p ^ q for p, q in zip(c, d))
    rev_A[key] = c
```

But this method is not perfect because the lenght of the dictionary afterwards is only `10_606_955` which is far from `256³ = 16_777_216`, so I updated my dictionary so that each key is paired with a list of values giving that key :

```python
rev_A = {}
md5 = _mh('md5')
for i in range(256 ** 3):
    c = i.to_bytes(3)
    d = md5(c)
    key = bytes(p ^ q for p, q in zip(c, d))
    if key in rev_A:
        rev_A[key].append(c)
    else:
        rev_A[key] = [c]

# same concept for rev_B
```

This is problematic because this will prevent an automatic decoding of `_T` if in practice `rev_A[key]` or `rev_B[key]` contains more than one value. To test it I started with the following reverse function :

```python
def rev(s, x):
    i = s.i
    x[i:i + 3] = rev_A[bytes(x[i:i + 3])][0]
```

## Final decoding

I tried to get the flag using this function but some characters were off :

```python
def get_flag():
    x = bytearray(_T)
    for o in reversed(_Q):
        o.rev(x)
    return bytes(x)
```

I added a print statement in the `rev` method of `_A` and `_B` to see if there is indeed an unicity problem and bingo :

```python
In [1]: get_flag()
rev B : [b'L\x94\xdb']
rev A : [b'y3^', b'\xa4Lf', b'\xe2\x83\x88']
```

By tweaking the index of the element in the list given by `rev_A` I found out that it was the last one and I could get the flag :

```python
In [3]: get_flag()
wreck{...}
```
