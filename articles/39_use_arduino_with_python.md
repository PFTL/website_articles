Using Python to communicate with an Arduino
===========================================

Arduinos may be one of the most disruptive electronic developments of the past decade. They opened a world of possibilities to quickly prototype solutions in professional settings but also opened the door for enthusiasts to learn about electronics and micro controllers. In this article we are going to see how to get started with an Arduino board and how to control it from a computer using Python. 

An Arduino is an electronics board packaging a micro-controller unit plus passive elements to ensure proper functioning, and headers to be able to interface with the real world very simply. But Arduinos are not just the electronics, they are also the ecosystem around them, including the software needed to program them as one sees fit.

In order to get started, we assume you have access to an Arduino, such as the Uno, but most of the available ones are going to work fine. The Nano, Micro, Due, Mega, they all have different specifications, but the way you program and interface with them is more or less the same. In this article we are not going to discuss about electronics and how to build circuits, but just about the software side, so you can follow along even if you don't have an Arduino board at hand. 


About the Arduino
=================

Some years ago, an Italian company launched a device that was set to
revolutionize the do-it-yourself (DIY) community: the Arduino. The
project was simple, they packaged known electronics into a very usable
format and provided the software needed for programming it. With this,
they opened the door to enthusiasts and professionals to tinker how to
interact with the real world. Arduinos are able to read analog signals,
digital inputs and also to generate outputs.

Over the years, several versions of the devices started to appear, each
with specific characteristics such as shape, connectivity, inputs and
outputs, etc. For the purpose of this tutorial, we are going to focus on
the Arduino Due, which has two analog outputs, a feature hard to find in
other devices. However, this particular board is very hard to find in
normal stores, you can of course get it from Aliexpress and Banggood.
Except for the analog output section, the code and the examples are the
same.

Installing the Arduino IDE
==========================

The best place to get started with the Arduino is the official
documentation. The first step is to install the editor that you need in
order to load code to the board. Head to
[Arduino.cc](https://www.arduino.cc/en/Guide/HomePage) and follow the
instructions according to your platform. Not only every operating system
is different, also different boards may have different requisites. For
example, for using the Arduino Due you need to include the USB drivers
in case you are working on Windows 7 or earlier.

Once you are done with the installation, it is time to start developing
the code for the board.

Acquiring Data
==============

This tutorial is not about Arduinos themselves but about how to build a
device that will allow you to acquire data from the computer, in exactly
the same way than what you would do if you had a DAQ card connected. In
any case, I will show you some basic examples for you to get acquainted
with the hardware and how to program it. If you open the Arduino IDE,
you will find an empty editor. Before starting to develop, you have to
check that both the type of card and the port are correct. Check the
image below to see where to find the options.

If you go through the menus, you will see that there are some examples
available, which can provide a very valuable starting point for learning
new options. In a broad sense, the code that runs in an Arduino can be
split in two parts: a setup process in which you initialize variables
and do some configuration, such as setting the parameters of some ports,
etc. This is run only once, when the device boots up. Then you have a
loop, in which all the interaction both with the real world and to the
computer happens.

Arduinos are not programmed with Python, but with C++ type of code.
Therefore the syntax is different, but the routines we will develop are
simple enough. If you are in doubt, the fastest is to learn by reading
code from others.

Downloading data with Python
============================

Structure the Arduino code to download data. Pyserial

Better Arduino Code
===================

Pyduino example

Python Driver
=============

Write a class
