Using Python to communicate with an Arduino
===========================================

Arduinos may be one of the most disruptive electronic developments of the past decade. They opened a world of possibilities to quickly prototype solutions in professional settings but also opened the door for enthusiasts to learn about electronics and micro controllers. In this article we are going to see how to get started with an Arduino board and how to control it from a computer using Python. 

An Arduino is an electronics board packaging a micro-controller unit plus passive elements to ensure proper functioning, and headers to be able to interface with the real world very simply. But Arduinos are not just the electronics, they are also the ecosystem around them, including the software needed to program them as one sees fit.

In order to get started, we assume you have access to an Arduino, such as the Uno, but most of the available ones are going to work fine. The Nano, Micro, Due, Mega, they all have different specifications, but the way you program and interface with them is more or less the same. In this article we are not going to discuss about electronics and how to build circuits, but just about the software side, so you can follow along even if you don't have an Arduino board at hand. 

## Getting Started With Arduino
If you have an Arduino or equivalent at hand, the first thing you must do is installing the Arduino IDE. You can download it from [here](https://www.arduino.cc/en/Main/Software). Even though you can develop code online, I strongly suggest you to have the tools on your computer, and use version control. For bigger projects, like the one we are going to build here, it is very handy to keep everything on the same place. 



- Arduino IDE
    - Installing
    - Connecting the board
- Simple program
    - Variable declaration (it is not Python)
    - Loop: Setup, Loop, delay, etc.
    - Serial Monitor
    - Triggering a measurement
    
## Reading the Arduino from Python
- PySerial
- Triggering a measurement (read/write=Query)
- PyVISA

## Better Arduino code
- Clearer read/write/output
    
## Driver with Python
- Building a class to interface with the Arduino

