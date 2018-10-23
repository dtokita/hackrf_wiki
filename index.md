# Lesson 01 - HackRF Introduction

## Setting Up

The software used to control the HackRF is called __GNU Radio Companion__. This tutorial run this on Windows.
If you decide to use Pentoo, do __NOT__ use a virtual machine.

Plug in the HackRF to your host computer, you should see the LEDs on the side of the device illuminate. This
indicates that the HackRF is getting power.

## Exploring the GUI

When opening the GNU Radio Companion for the first time, you should get something that looks like this:

![alt text](https://github.com/dtokita/hackrf_wiki/blob/master/pics/PIC1.PNG "GUI for GNU Radio Companion")

## Starting Off

The way that you can interact with blocks is by double-clicking them. Double-click on the __Generate Options__
block in the top left corner and change the property from __QT GUI__ to __WX GUI__.

We first want to create a source block which tells the HackRF to receive signals. This can be done by creating
a block called __osmocom Source__. This block can be found at __(no module specified)__->__Sources__->__osmocom Source__.
