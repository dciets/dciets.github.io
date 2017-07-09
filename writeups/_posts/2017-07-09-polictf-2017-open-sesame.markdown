---
layout: post
ctf: "PoliCTF 2017"
title: "PoliCTF 2017 - Open Sesame - Forensics"
---

>If you're a real hacker, then open my garage door!" said one of my friends.
>
>Challenge accepted.
>
>Note: the flag is not in the standard format (flag format: [01]+)
>
>Hint: MM53200

We are given a file named `gqrx_20170117_220606_433000000_8000000_fc.raw` which does not have a common header or known signature inside it. I googled for `gqrx` which lead to software to receive radio signal. I imported the file in Audacity as a raw audio file using the default settings. Altought the frequency was obviously off, we could see still clearly the data transmitted.

In the datasheet of the MM53200 chip, there's a diagram that shows how bits are encoded.

<img src="https://i.imgur.com/O6ZXSre.png">

A smaller silent gap means a 0 and a larger silent gap means a 1. Since the chip has 12 inputs, 12 bits are transmitted per pulse. This matches the waveform in Audacity. Here is the first pulse decoded.

<img src="https://i.imgur.com/dx4uDGG.png" />

There are a 8 pulses in the file and to find the one that is the actual garage door code, we have to back to the datasheet.

<img src="https://i.imgur.com/2OU86K7.png" />

The transmitter sends the code four times. On the receiver side, it will first check if the 12 bits received matches. If it does, it will expect 3 more pulses that also matches the 12 bits. Afterwards, the transmit/output goes low (which I assume makes the garage door open).

The first pulse is repeated 4 times which means it is the code to unlock the door. The flag is then one of the these pulses that decodes to `011010011000`.

Becojo - Northern Coalition
