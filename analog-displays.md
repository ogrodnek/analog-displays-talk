## Building Analog Displays for Your Data
### StrangeLoop 2014
### @ogrodnek
![](images/meters.jpg)

---


# Why?
fun!
engaging!
becomes part of your environment!
learn surprising things!
use other senses!
constraints drive creativity!

---

# Goals

- Inspire you to create your own analog displays
- Convince you it's _**easy**_!
- Give you a starting point

---

# Agenda

- hardware platforms overview/intro
- cover analog metric display projects
- next steps

---

# Platforms

- arduino
- spark core
- raspberry pi

---

## Let's talk Microcontrollers

![](images/microcontrollers.jpg)

---

# Arduino

![](images/uno.jpg)

^ 
- open source, cross-platform microcontroller platform
- community/examples
- in-expensive ($25, $15 clone)
- 8-bit, 16Mhz, 32K flash, 2K ram

---

## Hello World

![filtered](images/arduino-blink.gif)

---

# Blink!

```c
int led = 13; // Pin 13 has an LED connected on most Arduino boards.

// the setup routine runs once when you press reset:
void setup() {				
  pinMode(led, OUTPUT);    // initialize the digital pin as an output.
}

// the loop routine runs over and over again forever:
void loop() {
  digitalWrite(led, HIGH); // turn the LED on (HIGH is voltage level)
  delay(1000);             // wait for a second
  digitalWrite(led, LOW);  // turn the LED off by making voltage LOW
  delay(1000);             // wait for a second
}
```

^ 
- Can use C/C++
- Lots of examples, libraries

---

![fit](images/arduino-examples.jpg)

---

# Communication

Arduino has built-in USB serial

ethernet, wifi, bluetooth possible

---

# Firmata

Generic protocol for communicating with microcontrollers.

Client and device libraries

Makes it very easy to create _'dumb'_ devices.

Great for prototyping!


^ 
Load generic arduino sketch
easy to extend with custom functionality


---

# pyFirmata

https://github.com/tino/pyFirmata

```python
from pyfirmata import Arduino
import time

arduino = Arduino('/dev/tty.usbmodem1421')
led = arduino.get_pin('d:13:o')

while True:
  led.write(1)
  time.sleep(1)
  
  led.write(0)
  time.sleep(1)
```

^ 
specify USB serial device
digital pin 13, output

---

# Now you know

- how to blink an LED on an arduino
- how to make your _computer_ tell the _arduino_ to blink an LED

_**Go out and make a display!**_

---

# Spark Core
![](images/spark-core.jpg)


^
32-bit ARM Cortex
72 Mhz
128 flash, 20KB ram
$40

---

![100%](images/spark-ide.png)

---

![fit](images/spark-tinker-iphone.png)

---


# Spark Core Remote Blink

```bash
$ DEVICE="7adca5b8743743d0b4d257d0"
$ TOKEN="1eda6e8c08044613b8b5b29ce13a01e0"
$ curl https://api.spark.io/v1/devices/$DEVICE/digitalwrite \
>   -d access_token=$TOKEN \
>   -d params=D7,HIGH
{
  "id": "7adca5b8743743d0b4d257d0",
  "name": "blinky",
  "last_app": null,
  "connected": true,
  "return_value": 1
}

```

---


# Spark Core Custom Functions

```c
int brewCoffee(String command);

void setup() {
  Spark.function("brew", brewCoffee); // register
}

void loop() { }

// this function gets called upon a matching POST request
int brewCoffee(String command) {
  if (command == "coffee") {
    // do something here...
    return 1;
  }
  else return -1;
}
```

---


# Let's Brew!

```bash
$ curl https://api.spark.io/v1/devices/$DEVICE/brew \
>   -d access_token=$TOKEN \
>   -d params=coffee
{
  "id": "7adca5b8743743d0b4d257d0",
  "name": "blinky",
  "last_app": null,
  "connected": true,
  "return_value": 1
}
```

---

# Raspberry Pi

![](images/raspberry-pi.jpg)

^
$40, ARM 700Mhz, 512MB ram!
real OS! device support! ethernet, video, audio
GPIO!

---

## Pi Blink!

![](images/raspberry-pi-blink.gif)

---

## Blink Code

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BOARD)
GPIO.setup(7, GPIO.OUT)

while True:
    GPIO.output(7, True)
    time.sleep(1)
    GPIO.output(7, False)
    time.sleep(1)

```

---

# Let's talk displays!

---

![](images/meters-animated.gif)

^Analog meters
- great, classic look!
- would be better with custom faceplates!
- disappointing with cw 60 second resolution
- difficulty: LOW!

---

```python
import boto.ec2.cloudwatch
from pyfirmata import Arduino

arduino = Arduino('/dev/tty.usbmodem1421')
cpu_pin = arduino.get_pin('d:5:p')

cw = boto.ec2.cloudwatch.connect_to_region("us-east-1")

def cpu_utilization(name):
  return cw.get_metric_statistics(60, .., ..,
    'CPUUtilization', 'AWS/EC2', 'Average',
      {'AutoScalingGroupName': name})[0]["Average"]

while True:
    cpu = cpu_utilization("my-group-name")
    cpu_pin.write(cpu / 100)
    
    time.sleep(10)
```

^ 
arduino running 'StandardFirmata' example sketch
cw call truncated, otherwise full code
USB serial device
digital pin 5, PWM mode

---

![fit](images/pwm.png)


---

```python
import ...

GPIO.setmode(GPIO.BOARD)
GPIO.setup(7, GPIO.OUT)

cpuPin = GPIO.PWM(7, 50)
cpuPin.start(0)

cw = boto.ec2.cloudwatch.connect_to_region("us-east-1")

def cpu_utilization(name):
  return cw.get_metric_statistics(60, .., ..,
    'CPUUtilization', 'AWS/EC2', 'Average',
      {'AutoScalingGroupName': name})[0]["Average"]

while True:
    cpu = cpu_utilization("my-group")
    cpuPin.ChangeDutyCycle(cpu)
    time.sleep(10)
```

---

```python
import ...

access_token = "abcde1234"

user = healthgraph.User(session=healthgraph.Session(access_token))
records = user.get_records()

url = 'https://api.spark.io/v1/devices/012345678/analogwrite'
spark_token = "x12345"

for t, recs in records.items():
	if t == "totals":
		val = recs["Running"]["THIS_WEEK"]
		scaled = (val / 10) * 255
		d = urllib.urlencode({'access_token': token, 'params' : "A0,%.0f" % scaled})
		content = urllib2.urlopen(url=url, data=d).read()
		print content
```

---

![](images/exception-printer2.jpeg)

^ Exception printer
apps log exceptions to SNS, SNS->SQS, printer polls SQS
+ started as a joke - short order cook
interesting:
 - printer noise is a signal of activity.
 - size of trail indicator of system health

---

![](images/xylophone.jpeg)

^ 
computer controlled xylophone
solenoids (linear motor, electromagnetic)
different messages/events can play different note sequences
mostly a novelty, but, fun to play

---

![autoplay](videos/xylo-playing.MP4)

---

![](images/xylo-piano.jpg)

---

![](images/event-counters.jpeg)

^ Event Counters
electromechanical counter
only goes up!  no reset
not super useful for display...
BUT, makes great loud clicking noise when advancing -- you HEAR something is happening
think: train station

---

## future work?

---

![](images/instance-grid.jpeg)

^ Instance grid
work in progress
not sure it's useful
wanted to show breakdown by as group
 - lots of colors, but what's the key?
 - limited space, need lots of displays (# machines, regions)

---

![](images/other/etch-a-sketch.jpg)

---

# Next Steps

1. Get an Arduino
2. Blink an LED

---

# Shopping List

![](images/other/breadboard.jpg)

1. arduino/spark core/whatever
2. breadboard, jumper wires
4. LEDs! , meters , buttons
5. sensors
6. assorted resistors
7. multimeter?

---

# Outfitters

adafruit
sparkfun
evil mad scientist
mouser

---

![fit](images/soldering.png)

---


# PCBs

### software: gEDA, KiCad, eagle
### manufacture: OSH Park


![](images/pcb.png)


---

# Questions/Comments?

http://github.com/ogrodnek/analog-displays-talk

## Send me info on your analog displays!

ogrodnek@gmail.com
http://analogmachines.com/blog/

