# BreakfastSerial - Bluetooth serial WIP

A Firmata based framework for interacting with Arduinos over serial.
This fork is specific to getting bluetooth serial working.

## Hardware Setup

Not all Bluetooth serial devices are equal, you will most likely need to update the firmware on the device.
A good tutorial for that lives [HERE](http://www.instructables.com/id/Modify-The-HC-05-Bluetooth-Module-Defaults-Using-A/)

You will also want to use this to change the default from 9600 to 115200.

### Running on linux / raspberrypi

Install the modules needed:
``` bash 
 sudo apt-get install bluetooth bluez-utils blueman 
```

Pair with the device

Find bluetooth local device
``` bash 
 hcitool dev
 >Devices:
 >hci0	00:1A:7D:DA:71:0B
```


Find devices:
``` bash 
 hcitool scan
 > Scanning ...
 > 20:13:06:04:27:06	itead 
```


Pair the device:
``` bash 
 sudo bluez-simple-agent hci0 20:13:06:04:27:06
```

Make the device trusted
``` bash 
 bluez-test-device trusted 20:13:06:04:27:06 yes
```


To open the serial connection:
``` bash 
 sudo bluez-test-serial 20:13:06:04:27:06 &
```
Need to modify this in the future, as it currently opens the port of only 1000 seconds.

This will print out something like:
``` bash
> Connected /dev/rfcomm1 to 20:13:06:04:27:06
> Press CTRL-C to disconnect
```
 "/dev/rfcomm1" is the port we will use to connect to the arduino.
 This process need to be running in the back ground, in order for breakfast serial to connect to the port.


Need this for using the bluetooth module
``` bash
sudo apt-get python-bluez
```

## Arduino Setup

In order to use BreakfastSerial, you need to have an arduino running the
standard firmata.

1. Download the Arduino IDE from the arduino website
  - [OSX](http://arduino.googlecode.com/files/arduino-1.0-macosx.zip)
  - [Linux 32 bit](http://arduino.googlecode.com/files/arduino-1.0-linux.tgz)
  - [Linux 64 bit](http://arduino.googlecode.com/files/arduino-1.0-linux64.tgz)
  - Windows support coming soon.
2. Plug in your Arduino or Arduino compatible microcontroller via USB
3. Open the Arduino IDE, select: File > Examples > Firmata > StandardFirmata
4. Click the "Upload" button.

## Installation


#### From Source

``` bash
git clone git://github.com/kymwatts/BreakfastSerial.git && cd BreakfastSerial
python setup.py install

#grab theycallmeswift's pyfimata, for fixed bluetooth connection.
git clone git://github.com/theycallmeswift/pyFirmata.git && cd pyFirmata
python setup.py install
```

## Getting Started

The BreakfastSerial library provides a simple abstraction for a number of
common components.  Make sure your arduino is plugged in and is running firmata.

### Arduino

If you create a `Arduino` object without any parameters, it will attempt to auto discover 
the serial port that the Arduino is attached to and connect automatically.  Optionally,
you can supply the path to a serial port (Ex. `"/dev/tty.usbmodem4111"`).

``` python
from BreakfastSerial import Arduino
board = Arduino() # This will autodiscover the device
```

### Blink an LED

To use the led object, import Led from `BreakfastSerial`.  The constructor takes an
Arduino object and a pin number as its arguments.

``` python
from BreakfastSerial import Arduino, Led
from time import sleep

board = Arduino()
pin = 13
led = Led(board, pin)

led.on()
sleep(2)
led.off()
```

You can also use the `blink` method and pass it a number of milliseconds to automate the blinking process

``` python
millis = 200
led.blink(millis)
```

### Push a button

The `Button` component has a number of helper methods that make it easy to work with buttons.
The constructor takes an Arduino object and a pin number as its arguments.

``` python
from BreakfastSerial import Button, Arduino

board = Arduino()
button = Button(board, 8)

def down_cb():
  print "button down"

def up_cb():
  print "button up"

def hold_cb():
  print "button held"

button.down(down_cb)
button.up(up_cb)
button.hold(hold_cb)
```

The `down` and `up` functions are just nice wrappers around the underlying event emitter.  The `Button`
component emits the following events:

 - `change` - The button value changed
 - `down` - The button is pressed
 - `up` - The button is not being pressed
 - `hold` - The button was held for at least 1 second

### Use an RGB Led

The `RGBLed` component lets us change the colors of an RGB Led without having to
interact with the three underlying leds.

```python
from BreakfastSerial import Arduino, RGBLed
from time import sleep

board = Arduino()
led = RGBLed(board, { "red": 10, "green": 9, "blue": 8 })

led.red()
sleep(1)

led.green()
sleep(1)

led.blue()
sleep(1)

led.yellow()
sleep(1)

led.cyan()
sleep(1)

led.purple()
sleep(1)

led.white()
sleep(1)

led.off()
```

### LED Brightness

You can set the brightness of an LED with the `brightness` function.  The LED
must be on a PWM capable pin or it will throw and error.  Brightness is
measured on a scale of 0% to 100%.

```python
from BreakfastSerial import Arduino, Led
from time import sleep

board = Arduino()
pin = 9
led = Led(board, pin)

for x in range(0, 100):
  led.brightness(x)
  sleep(0.01)
```

### Read a sensor

The `Sensor` component lets us read in data from a sensor (analog or digital).  The constructor takes in
an Arduino object and a pin number.

``` python
from BreakfastSerial import Arduino, Sensor

board = Arduino()
sensor = Sensor(board, "A0")

def print_value():
  print sensor.value

sensor.change(print_value)
```

The `Sensor` object has the following properties:

 - `threshold` - the amount `value` must change by to trigger a `change`
   event (Default: `0.01`)
 - `value` - the value of the underlying pin

The `change` function is just a nice wrapper around the underlying event emitter.  The `Sensor`
component emits the following events:

 - `change` - The sensor value change by at least the amount of `threshold`

### Control a servo
 
The `Servo` component let's us control a servo.  The constructor takes in
an Arduino object and a pin number.
 
``` python
from BreakfastSerial import Arduino, Servo
from time import sleep

board = Arduino()
servo = Servo(board, "10")

servo.set_position(180)
sleep(2)
servo.move(-135)
sleep(2)
servo.center()
sleep(2)
servo.reset()
```

The `value` property of a `Servo` object is the current position of the servo in degrees

### Control a DC Motor
 
The `Motor` component let's us control a DC Motor.  The constructor takes in
an Arduino object and a pin number.  The motor must be on a PWM capable pin.
 
``` python
from BreakfastSerial import Arduino, Motor
from time import sleep

board = Arduino()
motor = Motor(board, 9)

motor.start(80)
sleep(2)
motor.speed = 50
sleep(2)
motor.stop()
```

The `speed` property is represented in a percentage of max speed.  So, `speed
= 80` is setting the motor to 80% speed.  Setting `speed` equal to `0` is the 
same as calling `stop()`.

### Moar!

There are a bunch of examples in the `examples/` folder.  Additional components
will be added over time, so be sure to check back regularly.

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/c3bf610a9c47da4e52190bf67b5d953c "githalytics.com")](http://githalytics.com/theycallmeswift/BreakfastSerial)
