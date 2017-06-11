---
layout: post
title:  Stratum-1 NTP Server
date:   2017-06-11 17:06:00 CET
categories:
---


# Stratum-1 NTP Server with a rubidium frequency standard

To build a stratum-1 NTP server with a stratum 0 source is maybe a hobby. I am sure in most cases it's not necessary to build a high precisely time server in the own network. For most IT infrastructure it's good enough to get the time via NTP protocol from Internet. But sometime we are ambitious and we build our own NTP server.

This project results out of an idea to be independent. I am already running stratum 1 servers in the own network. These are a very low frequency receiver based server and a GPS based NTP server. With this I am dependent of others. So could it be the political decision to switch off the system. This possibility is maybe small. But there could also be physical situations which prevents normal operation, for example during a solar storm. An other reason could be someone is using a jammer for a dedicated disruption of this service.

## Components to use

There is always the question about the costs. And therefore which time source can be used so that it is not too expensive. A rubidium frequency standard you can buy nowadays (2017) for about 300 EUR in the second-hand market at Ebay. My decision was to buy an EFRATOM LPRO-101 as time source. The server should also be  cheap. Therefore I decided to use a Banana Pi M1. At that time when I started this project (it's about 2 years ago) the M1 was the only with a 1GB interface. Of course a short latency is always good for NTP. And the operating system has to be a Linux tpye.

## How-to build such a server

With a hardware like the Banana Pi or a Raspberry Pi one has already an interface called GPIO. Source code of NTP classic or NTPsec has a driver which can handle an 1PPS (one pulse per second) signal. The rubidium oscillator delivers a 10 MHz sinus wave. To get a 1PPS you need a divider by 10^7. All together 3 components.

* Banana Pi M1 micro computer with Linux
* 10 MHz oscillator EFRATOM LPRO-101
* divider by 10^7

And of course power supplies for these devices.

The Banana Pi M1 and the EFRATOM I could buy. The divider I had to develop by myself. I put this product at GitHub. You can find the source for this PC-board here

[https://github.com/hans-mayer/teiler10e7](https://github.com/hans-mayer/teiler10e7)

## More details

Coming soon.
