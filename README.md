# gbit-poe-splitter

Power devices which don't support PoE and fine-adjust the received voltage at the same time. This equipment conforms to the *802.3at* standard, also known as *PoE+*.

⚠️ This project is far from being done, and I'm just planning things out at this point in time.

## Table of Contents

* [Goals](#goals)
* [Introduction](#introduction)
  * [802\.3af (PoE)](#8023af-poe)
  * [802\.3at (PoE\+)](#8023at-poe)
  * [802\.3bt (Hi\-PoE, PoE\+\+)](#8023bt-hi-poe-poe)
* [Concept](#concept)
* [Features](#features)
  * [Ports](#ports)
  * [Settings](#settings)
  * [Status LEDs](#status-leds)
* [Parts](#parts)
* [Prototyping](#prototyping)

## Goals

* ✅ Small footprint PCB
* ✅ Max budget of 15 Euros
* ✅ Support up to 60V of input
* ✅ Selectable output voltage
* ✅ Selectable PoE class
* ✅ Highest possible current throughput
* ✅ Indication of active type 2
* ✅ DC Output Jack
* ✅ Proper centertap-termination

## Introduction

PoE is short for *Power over Ethernet* and will allow to transmit the needed power for devices on the other end of the CAT-X cable. This is especially of interest for already existing infrastructure, where it's just not economical to run a separate power wire. The only drawbacks are that there's a limit to how much energy can be transmitted and that you might experience a voltage-drop and thus an energy-loss over longer distances of wire.

### 802.3af (PoE)

This standard adds power-delivery to *802.3*, with up to *15.4W* at the output of the power sourcing equipment (PSE), using two twisted-pairs for injection. After calculating the average real world losses over the wire , the consumer only receives around *12.95W* of power. There are also lower power-classes, which are not the default and need to be registered for by the powered device (PD) explicitely.

| Class | Min Power (at PD, [W]) | Max Power (at PD, [W]) |
|:------|:----------|:----------|
| 0 | 0.44 | 12.95 |
| 1 | 0.44 | 3.84 |
| 2 | 3.84 | 6.49 |
| 3 | 6.49 | 12.95 |

### 802.3at (PoE+)

Since *802.3af* is not sufficient for some standard usecases, *802.3at* bumps it up a notch, while still using only two twisted-pairs. Now, PDs can receive up to *25.5W*, which should be plenty for switches, access-points, VOIP, card-readers and so on and so forth.

| Class | Min Power (at PD, [W]) | Max Power (at PD, [W]) |
|:------|:----------|:----------|
| 4 | 12.95 | 25.5 |

### 802.3bt (Hi-PoE, PoE++)

In order to meet the higher demand of power in special cases, IEEE came up with this standard at the end of 2018. Instead of two pairs, it uses all four pairs for power delivery, which will allow up to **100 watts**. Power loss gets cut in half, at least in theory. Since this standard requires more expensive technology, it's not covered by this project.

## Concept

There are several [requirements](https://www.ieee802.org/3/poep_study//email/pdf00005.pdf) to the PD, if it wants to consume *802.3at*.

The main component is represented by the [H6062NL](https://eu.mouser.com/ProductDetail/Pulse-Electronics/H6062NL?qs=hlTQ6wBu%252Bnnq8WII%2F92SGQ%3D%3D), which is a signal transformer that supports up to *1000Base-T*. It consits of four individual isolation transformers, which offer a center-tap for power delivery purposes, where one transformer corresponds to one twisted pair, to fully isolate the PD.

In order to support detection, classification, inrush-current limiting and short-circuit protection, I decided on the [TPS2379](https://www.ti.com/lit/ds/symlink/tps2379.pdf?ts=1636211997743&ref_url=https%253A%252F%252Fwww.google.com%252F). It supports up to *1A* of operating current, which should be plenty, and can even be extended using an external mosfet.

To tap the available power from either one of the two pairs, and to handle reverse-polarity situations, two full bridge rectifiers are inserted directly inbetween the transformers and the control-IC. The output voltage will then be fed through a buck-converter, to easily regulate it. [LM2596HV](http://hmsemi.com/downfile/LM2596HV.PDF) seems to be a perfect fit for this job.

## Features

### Ports

* Ethernet in (from PSE)
* Ethernet out (to non-PD)
* Standard DC-jack to supply power

### Settings

* 5 positions for class 0-4
* Voltage regulation potentiometer

### Status LEDs

* Power available (Green)
* Inrush current limiting (Yellow)
* Type 2 active (Yellow)
* Pair 1 & 2 hot (Red)
* Pair 3 & 4 hot (Red)

## Parts

| Part | Description | Shop | Price/Pice | Amount | Total |
|:-----|:------------|:-----|:-----------|:-------|:------|
| **Integrated Circuits** |
| H6062NL | Signal Transformer | Mouser | 4.27€ | 1 | 4.27€ |
| TPS2379 | 802.3af PD Interface | Mouser | 2.35€ | 1 | 2.35€ |
| MB1S-TP | Bridge Rectifier | Mouser | 0.26€ | 2 | 0.52€ |
| LM2596HV | [Regulator & aux. Components](https://de.aliexpress.com/item/32916300929.html?spm=a2g0s.9042311.0.0.55414c4dqviJXF) | AliExpress | 1.12€ | 1 | 1.12€ |
| BAS21LDYL | GP Diode | Mouser | 0.20€ | 2 | 0.40€ |
| SI7454FDP-T1-RE3 | N-Channel Mosfet | Mouser | 0.69€ | 1 | 0.69€ |
| **Connectors** |
| RJLSE4118101 | RJ45 8P8C Shielded | Mouser | 0.91€ | 2 | 1.82€ |
| PRT-10811 | 5.5mm DC Jack | Mouser | 0.81€ | 1 | 0.81€ |
| **Status LEDs** |
| RC0603FR-071K91L | 1.91k Ohm 0603 | Mouser | 0.008€ | 5 | 0.032€ |
| 150060GS75000 | LED Green 0603 | Mouser | 0.12€ | 1 | 0.12€ |
| 150060GS75000 | LED Yellow 0603 | Mouser | 0.12€ | 2 | 0.24€ |
| 150060GS75000 | LED Red 0603 | Mouser | 0.12€ | 2 | 0.24€ |
| **Filtering & Termination** |
| RC0603FR-0775RL | 75 Ohm 0603 | Mouser | 0.006€ | 8 | 0.012€ |
| CL10B103KO8NNNC | 10nF 0603 | Mouser | 0.011€ | 4 | 0.044€ |
| C0805C104K8RACAUTO | 100nF 0805 | Mouser | 0.058€ | 1 | 0.058€ |
| CL10C102JB8NNND | 1nF 0603 | Mouser | 0.016€ | 2 | 0.032€ |
| **PoE Class (R_CLS)** |
| RC0603FR-071K27L | 1.27k Ohm 0603 | Mouser | 0.008€ | 1 | 0.008€ |
| CR0603-FX-2430ELF | 243 Ohm 0603 | Mouser | 0.016€ | 1 | 0.016€ |
| ERJ-3EKF1370V | 137 Ohm 0603 | Mouser | 0.014€ | 2 | 0.014€ |
| CR0603-FX-90R9ELF | 90.9 Ohm 0603 | Mouser | 0.013€ | 1 | 0.013€ |
| CR0603-FX-63R4ELF | 63.4 Ohm 0603 | Mouser | 0.013€ | 1 | 0.013€ |
| SX1100-A | Jumper Shunt | Mouser | 0.03€ | 1 | 0.03€ |
| 54202-T0803AB01LF | Jumper Pins 2x5 | Mouser | 0.65€ | 1 | 0.65€ |
| **R_DEN** |
| RK73H1JTTDD2492F | 24.9k Ohm 0603 | Mouser | 0.01€ | 1 | 0.01€ |
| **Miscellaneous** |
| RK73B1JTTD204J | 200k Ohm 0603 | Mouser | 0.008€ | 1 | 0.008€ |

Current total: 13.51€, not **yet** including: PCB material

## Prototyping

07-12-2021:

I wanted to check if I could extract power using only the transformer, like some people managed to do online, with no additional logic. After finding an old gigabit-capable router in my basement, I disassembled it and unsoldered one of it's internal transformators ([G4802CG](https://media.digikey.com/pdf/Data%20Sheets/Mentech%20PDFs/G4802CG.pdf)). I also tried one from a 100M model, which didn't work, because they've joined up the centertaps on neighboring coils. The pins were so small that a breakout-board was necessary. This was my test-setup:

![Breakout-Board](img/breakout_board.jpg)

On the left, the PSE is connected, where the power gets extracted, and on the right, it's my laptop (PD). I'm using a `Netgear GS305P`, and nope, it's not directly spitting out power without additional logic. I actually discovered that it pulses a few volts on all ports, no matter if something is connected or not. I guess that's where the PD would use load modulation to register itself and it's desired power-class. Now I'm waiting to receive my mouser-order to continue with this test-setup, until the first prototype is up and working, only then will I create a full PCB-layout.

Just in case you're wondering, here's whats inside the IC, I think it looks wonderful:

![Transformer Inside](img/transformer_inside.jpg)