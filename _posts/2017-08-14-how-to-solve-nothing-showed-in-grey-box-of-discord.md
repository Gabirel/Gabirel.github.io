---
layout: post
lang: en
title: How to solve nothing showed in grey box of Discord
categories: solution discord
tags: tips discord
date: 2017-08-14
---

<p class="intro"><span class="dropcap">W</span>e all know that what discord is and what it does. Okay, if you don’t know what it is, just skip this post.</p>

This post is only for those who have showing issue in their grey box.

<center><blockquote>Platform: Linux Arch-like</blockquote></center>

If other distros have the same issue, you may try this. Don’t know whether it works for you.

-----
Table of Contents

* TOC
{:toc}

-----



# Issue

If you update your discord to `Linux-v0.0.2`, you should encounter this issue that nothing shows in the grey box of Discord below.

<center>
  <img src="/assets/img/solve-discord-grey-box/discord-blank.png" alt="Nothing shows in grey box">
</center>

Okay, this is easy.

Install `libc++`.

In Arch/Manjaro, use this:

```shell
$ yaourt -S libc++
```

# Another problem when trying to install libc++

Another problem:

## Issue

As you can see the pic below:

<center>
<img src="/assets/img/solve-discord-grey-box/discord-blank-error.png" alt="PGP signatures could not be verifed!">
</center>

## Solution

Execute the following to import keys using gpg:

```shell
$ gpg –recv-keys <KEYID - See ‘validpgpkeys’ array in PKGBUILD>
```

# Why need libc++

As you can see in libc++ in [AUR](https://aur.archlinux.org/packages/libc%2B%2B/), libc++ is required by discord.

<center>
<img src="/assets/img/solve-discord-grey-box/why-discord-need-this.png" alt="libc++ is requried by discord">
</center>


Actually, it’s really weird that Discord needs libc++ at `v0.0.2`. It doesn’t need libc++ before.

Anyway, it should do the tricks.

<center><blockquote>The End</blockquote></center>
