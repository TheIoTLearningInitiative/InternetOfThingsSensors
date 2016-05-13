# Hands-On

Here we are going to experiment  how to communicate with a RGB LCD Display (http://www.seeedstudio.com/wiki/Grove_-_LCD_RGB_Backlight) and a barometric sensor (http://www.seeedstudio.com/wiki/Grove_-_Barometer_Sensor) from Grove, which by the way use I2C  as  communication protocol.

We will start with baby steps to later throw away UPM & MRAA:

* <font color="blue">Using libraries (UPM & MRAA)</font>
  * Shows how kids and not so hardcore nerds communicate with sensors
* <font color="blue">Look Mama! without Handles! (writing our own sensor code in user space)</font>
  * Teach you how to use user space i2C  libraries to communicate directly with your device.
* <font color="blue">I'm a pro! (turning our sensor code into a module)</font>
  * We have gone way beyond from the comfort zone, lets get into the Kernel
