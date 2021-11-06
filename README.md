# gbit-poe-splitter

Power devices which don't support PoE and fine-adjust the received voltage at the same time. This equipment conforms to the *802.3at* standard, also known as *PoE+*.

⚠️ This project is far from being done, and I'm just planning things out at this point in time.

## Table of Contents

* [Goals](#goals)
* [Main Parts](#main-parts)
* [Introduction](#introduction)
  * [802\.3af (PoE)](#8023af-poe)
  * [802\.3at (PoE\+)](#8023at-poe)
  * [802\.3bt (Hi\-PoE, PoE\+\+)](#8023bt-hi-poe-poe)
* [Concept](#concept)

## Goals

* ✅ Small footprint PCB
* ✅ Max budget of 12 Euros
* ✅ Support up to 60V of input
* ✅ Selectable output voltage
* ✅ Selectable PoE class
* ✅ Highest possible current throughput
* ✅ Indication of active type 1/2
* ✅ DC Output Jack
* ✅ Proper centertap-termination

## Main Parts

| Part | Description | Shop | Price/Pice | Total |
|:-----|:------------|:-----|:-----------|:------|
| H6062NL | Signal Transformer | Mouser | 4.27€ | 4.27€ |
| TPS2379 | 802.3af PD Interface | Mouser | 2.35€ | 2.35€ |
| SI7454FDP-T1-RE3 | N-Channel Mosfet | Mouser | 0.69€ | 0.69€ |
| MB1S-TP | Bridge Rectifier | Mouser | 0.26€ | 0.52€ |
| SS-90000-00 | Barebones RJ45 | Mouser | 0.47€ | 0.47€ |
| PRT-10811 | 5.5mm DC Jack | Mouser | 0.81€ | 0.81€ |
| LM2596HV | Regulator & aux. Components | AliExpress | 1.12€ | 1.12€ |

Current total: 10.23€, not **yet** including: PCB, resistors, capacitors, leds.

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