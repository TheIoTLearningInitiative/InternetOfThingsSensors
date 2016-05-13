# Hands-On

Here we are going to experiment  how to communicate with a RGB LCD Display (http://www.seeedstudio.com/wiki/Grove_-_LCD_RGB_Backlight) and a barometric sensor (http://www.seeedstudio.com/wiki/Grove_-_Barometer_Sensor) from Grove, which by the way use I2C  as  communication protocol.

We will start with baby steps to later throw away UPM & MRAA:

* <font color="blue">Using libraries (UPM & MRAA)</font>
  * Shows how kids and not so hardcore nerds communicate with sensors
* <font color="blue">Look Mama! without Handles! (writing our own sensor code in user space)</font>
  * Teach you how to use user space i2C  libraries to communicate directly with your device.
* <font color="blue">I'm a Pro! (turning our sensor code into a module)</font>
  * We have gone way beyond from the comfort zone, lets get into the Kernel

<br>
<br>
###Using libraries (UPM & MRAA)
Do not misunderstand me , UPM and MRAA are great libraries, they really make things easier, specially when you are starting and you would like to get a prototype up and running fast, instead of struggling in how to get the values from your sensors and devices.

So first let's recap. MRAA is used to communicate to the device through I2C, PWM,SPI, etc.. on top of that UPM is used so at the end you just have to instantiate an object of the class of your device or sensor. lets see how we can do that:

####1. Check if the sensors are supported by MRAA
At the beginning we stated that we are going use the LCD RGB  Backlight Display and the Barometric sensor, please go to http://iotdk.intel.com/docs/master/upm/modules.html and find out either using the search on the top right corner or manually browse the sensor list to see if our sensors are supported by MRAA.



####Look Mama! without Handles! 

####I'm a Pro!
