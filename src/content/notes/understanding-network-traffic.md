---
title: Understanding network traffic
description: What's actually moving across the wire, and how to read it — from a capture to a first detection idea.
date: 2026-06-16
tags: [networking, pcap, blue-team]
draft: true
---

Packets are one of those things you can hand-wave for a while, until you can't. These are my notes on actually reading what moves across the wire — and the first thing I'd watch for as a defender.

## 01 · Why I'm doing this

[ Write: what you couldn't do before, and what you want to be able to do after. ]

## 02 · The setup

[ Write: tools and where you captured. ]

**Tools:** tcpdump / Wireshark. **Source:** your own machine is the safe sandbox.

## 03 · Capturing the traffic

[ Write: the exact command + what each flag did. ]

```bash
# replace with what you ran
sudo tcpdump -i any -n -c 50
```

## 04 · Reading one packet

[ Write: walk one packet end to end — layers, ports, the conversation. ]

## 05 · What surprised me

[ Write: the thing that didn't match your mental model. ]

## 06 · A detection angle

[ Write: as a defender, what reads normal vs. worth a look? ]

## 07 · Notes to self

[ Write: loose ends, next thing to dig into. ]
