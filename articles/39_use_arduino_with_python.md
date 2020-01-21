Using Python to communicate with an Arduino
===========================================

Arduinos may be one of the most disruptive electronic developments of the past decade. They opened a world of possibilities to quickly prototype solutions in professional settings but also opened the door for enthusiasts to learn about electronics and micro controllers. In this article we are going to see how to get started with an Arduino board and how to control it from a computer using Python. 

An Arduino is an electronics board packaging a micro-controller unit plus passive elements to ensure proper functioning, and headers to be able to interface with the real world very simply. But Arduinos are not just the electronics, they are also the ecosystem around them, including the software needed to program them as one sees fit.

In order to get started, we assume you have access to an Arduino, such as the Uno, but most of the available ones are going to work fine. The Nano, Micro, Due, Mega, they all have different specifications, but the way you program and interface with them is more or less the same. In this article we are not going to discuss about electronics and how to build circuits, but just about the software side, so you can follow along even if you don't have an Arduino board at hand. 

The main goal of this tutorial is to show you how you can communicate with an Arduino using Python. This involves both programming the Arduino in a clever way, and understanding some basic Python libraries which will allow you to build solutions very quickly. 

## Getting Started With Arduino
If you have an Arduino or equivalent at hand, the first thing you must do is installing the Arduino IDE. You can download it from [here](https://www.arduino.cc/en/Main/Software). Even though you can develop code online, I strongly suggest you to have the tools on your computer, and use version control. For bigger projects, like the one we are going to build here, it is very handy to keep everything on the same place. 

After you install the program, just run the Arduino IDE. Note that the interface is very minimalistic but does exactly what it was designed to do. Connect your board to the computer. Click on Tools/Ports and you should see your board listed there. Note that in principle you could have several boards connected at the same time, and if you plug/unplug the board from the computer, the port may change. It is always wise to check whether the port is properly configured. 

!!! warning
    If you are on a Linux computer, you will need to add the appropriate permissions to work with the USB ports. You can follow [this guide](https://www.arduino.cc/en/guide/linux)

## The first Arduino program
We are going to build a very simple program that is able to read an analog signal. We are choosing this scenario because most boards are able to read an analog input. Moreover, analog inputs are susceptible to noise and this means the value can change if you approach your hand to it, touch it with a wire, etc. Digital ports, since they only produce one of two values, are a bit more resilient to this kind of testing. 

The first challenge a Python programmer faces when programming an Arduino is that the language is different. The Arduino IDE works with a language inspired by C and C++. It is not hard to understand, and there are many examples you can use as a starting point. Also, the programs you develop inside an Arduino are tightly coupled with the real world through sensors and actuators, and this adds a new layer of complexity. 

## The Arduino Program Loop
As soon as you start the Arduino IDE, you will find an almost blank file, in which to functions are defined:

```c
void setup() {
  // put your setup code here, to run once:
}

void loop() {
  // put your main code here, to run repeatedly:
}
```

Arduino programs have two very distinctive stages: the **setup** stage is where you initialize whatever needs to be initialized, such as serial communication, while the **loop** is running continuously. So, imagine you want to switch on and off an LED, you would develop that piece of code in the **loop**. 

## Switching on and off an LED (Blinking)
we are going to use the built-in LED of an Arduino UNO, but the majority of the other boards are going to work as well. The first step is to define the digital pin to which the LED is connected as an output pin. Since we need to do this only once, at the beginning of the program, we do it on the ``setup`` function:

```c
void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}
```

You see here that we are using two variables: ``LED_BUILTIN`` and ``OUTPUT`` which were not defined anywhere. They are Arduino built-in constants, you can see more on the [documentation](https://www.arduino.cc/reference/en/). The next step is to switch on and off the LED, and we can do that on the ``loop```function:

```c
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(1000);
  digitalWrite(LED_BUILTIN, LOW);
  delay(1000);
}
```

In the loop, what we are doing is setting the output digital pin to ``HIGH``, which means the LED will switch on. We wait for 1000 milliseconds and switch it off. Then it starts again, hence the name ``loop``. You can upload the code to your Arduino, and once its done, you will see an LED on the board blinking. 

It is very important to note at this stage that there is no need to have a computer connected to the board. After you upload the program, you can power the Arduino with a USB charger and the LED will still blink. This is one of the nicest features of micro-controllers and is that they are able to run on their own, and normally they are much less error-prone than entire computers. 

## Monitoring the status of the board
The next step would be to be able to monitor from the computer whether the LED is on or off. Perhaps you would like to know whether it is working fine, or you would like to count the number of times it switches on, etc. For this, we will need to change a bit the code from the previous section. 

The first step is to think how can the Arduino send information to the computer. As you may have already noticed, we used the USB port to load the program to the board, and thus, it is reasonable to think we can use the same connection to get data from the device, and you are right. In order to be able to send information we will need to use the ``Serial`` communication. First we establish the communication:

```c
void setup() {
  Serial.begin(19200);
  pinMode(LED_BUILTIN, OUTPUT);
}
```

Note here that the argument of the ``begin`` function is known as the baudrate. This basically fixes how fast data is transferred, and only few values are acceptable, and ``9600`` is perhaps the most common one. Remember the number you put in there since it will appear also later. We are now ready to send information through the serial to the computer:

```c
void loop() {
  Serial.write("Switching ON");
  digitalWrite(LED_BUILTIN, HIGH);  
  delay(1000);
  Serial.write("Switching OFF");
  digitalWrite(LED_BUILTIN, LOW);
  delay(1000);
}
```

Again, you can upload the program to the board and you will see that the LED still blinks. So, how can we actually see the information being printed to the serial communication? In the Arduino IDE open Tools/Serial Monitor. And now you should see that every time the LED changes from on to off or the other way around, there is a message appearing on the screen. 

!!! warning 
    At the bottom of the serial monitor window, be sure you select the proper baud rate, or the messages are going to be nonsense.

Of course, you are probably also seeing that the messages get printed one after the other. This is because in serial communication there is no specific way of saying *this message has ended*. Something that is sort of a standard is using special characters to mark the end of a message, such as the newline character. Such as with Python, newline is represented by ``\n``, so you can update the code to include it, for example:

```
Serial.write("Switching ON\n");
```

This discussion is important, because we are not forced to use the new line character, we could have used anything we wanted. If you pay attention to the serial monitor window, you see there are some options regarding the line ending. We are going to keep the ``\n`` but be aware that other programs and other hardware may use a different character.  

## Controlling the LED
Right now the LED is switching independently from what we do with the computer. So now it is time to be able to control the LED at our own will from the computer. And probably you have anticipated it already, we are going to use the ``Serial`` communication, but reading from it instead of just writing to it. Before going through a lengthier discussion on how to do things properly, let's quickly make a ping-pong program, i.e. one that replies with the same message we send:

```c
int val;

void setup() {
  Serial.begin(19200);
}

void loop() {
  if (Serial.available() > 0){
    val = Serial.read();
    Serial.write(val);
    Serial.write("\n");
  }
}
```

What you have to pay attention to is that we first define ``val`` as an integer. In the ``loop``, we first check whether there is any serial data available. If there is, we read from it and write it back. We also write a newline, so things appear nicely formatted. 

If you open the serial monitor, you can now send messages to your device, and it will write them back to you. There are several things to notice here, first, what happens if you send a character such as an ``a`` instead of a number? Surprisingly, you get the same character back. Even though ``val`` is an integer, it gets converted to a character when writing on the serial. 

The second thing you have to see is what happens if you write several letters and then you press enter. You will see that every character gets a new line. This means that the ``read`` method is getting byte by byte. You can try a lot of different things, such as what happens if you send non-ascii characters, if you put delays, etc. The fact that is so easy to play around with all the parameters can help you understand how serial communication works. 

### Controlling the LED
So, we know how to send a message to the Arduino and how to do something with the received message. We must now go one step further and change the LED to on or off based on the message we send. Let's quickly see a possible program to achieve it:

```c
int val;

void setup() {
  Serial.begin(19200);
}

void loop() {
  if (Serial.available() > 0){
    val = Serial.read();
    Serial.write("\n");
    if(val==1){
      Serial.write("ON\n");
      }
    if(val==0){
      Serial.write("OFF\n");
      }
  }
}
```

The program above will only write back to us whether we are switching on or off through the serial communication. We do this because it makes it very easy to debug. If something is not working we can check whether is the program or the electronics that are giving problems. After you upload the code above, and you try it, you will notice that it is not working. Can you guess why? 

We have seen earlier that the serial read gets bytes out of the communication. This means, that if we send a ``1``, it will be transformed into bytes, which will be received by the Arduino and then transformed back into a 1. It is not worth pending too much time on this topic at this stage, but it all relates to encoding characters. You can take a look at the [ascii table](https://www.asciitable.com/) and notice that if you encode the character ``1``, you will get the bytes corresponding to the number 49, i.e.: ``110001``. 

The serial communication works by actually sending bite by bite of information in the form of a square wave. If the square wave is up corresponds to a 1, if it stays low, it is a 0, and so forth. Of course, to build a byte you need to have a fixed number of bits, or we would have problems knowing when one byte finished and the next started. Anyways, if we change the program above, we can use the information on the ascii table:

```c
void loop() {
  if (Serial.available() > 0){
    val = Serial.read();
    Serial.write("\n");
    if(val==49){
      Serial.write("ON\n");
      }
    if(val==48){
      Serial.write("OFF\n");
      }
  }
}
```
If you test it, you will see that this is working fine. If you send a 1, the serial will get the ``ON`` message, and the opposite if you send a 0. However, is this useful? I would argue strongly against it since it is very hard to understand the comparison to ``49`` and to ``48``. The best would be to change the value you are reading into a string, or an integer which we can compare. For simplicity let's just focus on converting to an integer, we just need to change the line in which we read and the comparison lines:

```c
void loop() {
  if (Serial.available() > 0){
    int val = char(Serial.read())-'0';
    Serial.println(val);
    Serial.write("\n");
    if(val == 1){
      Serial.write("ON\n");
      }
    if(val == 0){
      Serial.write("OFF\n");
      }
  }
}
```
As you see, the way in which we change from bytes to an integer is slightly convoluted. We exploit the fact that characters are essentially numbers, and that they are in order on the ascii table. If we subtract the 0, we will start counting at the proper position. Nonetheless, the code shown above works. If we send a 1 to the arduino we get the proper message back. To switch ON/OFF the LED, we just need to add the proper instructions. The final code would look like this:

```c
void setup() {
  Serial.begin(19200);
  Serial.flush();
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  if (Serial.available() > 0){
    int val = char(Serial.read())-'0';
    Serial.println(val);
    Serial.write("\n");
    if(val == 1){
      Serial.write("ON\n");
      digitalWrite(LED_BUILTIN, HIGH);  
      }
    if(val == 0){
      Serial.write("OFF\n");
      digitalWrite(LED_BUILTIN, LOW);
      }
  }
}
```
Now you are effectively switching ON/OFF an LED from the computer. I don't know about you, but I find this incredibly rewarding. Probably your head is already branching out to different possibilities. There is only one more topic to cover before we move into Python. 

## Reading a signal
Arduinos allow you to interact with the world not only by setting parameters, such as an LED, on or off, but also by reading values from the environment. Normally, you would need some sensor in order to make a valuable reading, but we are just going to read noise, so we don't worry about having a sensor at hand, and without worrying about going out of range. Let's quickly see how to get an analog signal:

```
int analogPin = A0;
int val = 0;  

void setup() {
  Serial.begin(19200);
}

void loop() {
  val = analogRead(analogPin);
  Serial.println(val);
}
```
The only part of the code which may not be self-explanatory is the first line. ``A0`` represents the input port we are reading, in this case is the analog port number 0. Almost all Arduino boards have analog inputs to read, the amount, however, can change. You can run the program above and see that there are a lot of numbers appearing on screen. 

If you move the board, and especially if you touch the pins from below, you will see that these numbers can change. Interesting, don't you think? It is up to you, at this stage, to come with an explanation on why this happens. 

Probably you noticed also that we are using ``println`` instead of ``write``. This is handy because we can then forget about the line ending. We are almost done with the introduction to the arduino, we have just to cover a detail.

## Triggering a measurement
Up to now, we used the serial input just to switch the LED on or off. We can use exactly the same approach to read the analog channel. It really takes no time to implement, the full code would be:

```c
int analogPin = A0;
int val = 0;

void setup() {
  Serial.begin(19200);
  Serial.flush();
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  if (Serial.available() > 0) {
    int val = char(Serial.read()) - '0';
    if (val == 1) {
      Serial.write("ON\n");
      digitalWrite(LED_BUILTIN, HIGH);
    }
    if (val == 0) {
      Serial.write("OFF\n");
      digitalWrite(LED_BUILTIN, LOW);
    }
    if (val == 2) {
      val = analogRead(analogPin);
      Serial.println(val);
    }
  }
}
```

Now, if you send a ``1``, the LED switches on, off with a ``0```, and if you send a ``2``, you will get the reading from the analog port number 0. We are all set to go to hook it up to Python. 

## Reading the Arduino from Python
We know how to send command from the serial monitor of the Arduino IDE, but this may be very limiting. Let's see how can we control our Arduino board using Python. In order to control devices, we will need some libraries installed. Perhaps you noticed that we have always been talking about serial communication. Serial is an old standard, which is normally associated with the bulky [RS-232](https://en.wikipedia.org/wiki/RS-232) connectors. Even though the connectors have fallen in disgrace, the protocol for sharing data is as alive as always was. The trick is that the arduino has a built-in chip that transforms the USB port into a Serial device. Little trick that saves a lot of effort when programming the boards. 

So, let's first get the basic library we need, **pyserial**:

```bash
pip install pyserial
```

This is the basic building blog for communicating with serial devices on Python. Be extra careful because there is another library called **serial** which has nothing to do with serial communication. 

Now that we have it installed, we can test it, by making it list all the serial devices connected to our computer:

```bash
python -m serial.tools.list_ports
```
 
 The output depends on your operating system, if you are on Windows, this is what you would get something like:
 
```bash
COM4
1 ports found
```

If you are on Linux or Mac, something like:

```bash
/dev/ttyACM0
1 ports found
```
The name of the port you are getting is the same you should have used when developing with the Arduino IDE. However, it may also be that if you plug/unplug the board, reboot the computer, etc. it changes. Note the port name, since we are going to use it extensively, and you will need to replace it on your own code. 

Let's move to a Python script now, that allows us to switch on and off the LED. We start by importing the needed libraries and opening the communication with the device:

```python
import serial

dev = serial.Serial("COM4", baudrate=19200)
```

Two important things in the simple code above. First, you should replace ``COM4`` by the serial port you got earlier. The second, the baudrate is given by the Arduino program we developed in the previous sections. We could have used 9600, and there is no a-priory way of knowing. You are left to the documentation of the hardware in order to know what is the proper baudrate to communicate. Not even the serial monitor is able to pick it up automatically. 

And now, to the juicy part. Let's switch on and off an LED:

```python
from time import sleep

dev.write(b'1')
sleep(1)
dev.write(b'0')
sleep(1)
dev.write(b'1')
``` 
I know, I know, could have been much much better! You can have a for loop, accept input, etc. You probably remember that every time we were setting the LED, there was a message confirming it, right? So we can also quickly read from the Arduino:

```python
dev.write(b'1')
print(dev.readline())
sleep(1)
dev.write(b'0')
print(dev.readline())
sleep(1)
dev.write(b'1')
print(dev.readline())
``` 

Now, at this stage is when things get really messy. Are you seeing the output you are expecting? Probably not. Especially if you updated the code, but never rebooted the Arduino. This happens because messages accumulate on a buffer, and you read whatever is available. Antoher thing that can happen



- Triggering a measurement (read/write=Query)
- PyVISA

## Better Arduino code
- Clearer read/write/output
    
## Driver with Python
- Building a class to interface with the Arduino

