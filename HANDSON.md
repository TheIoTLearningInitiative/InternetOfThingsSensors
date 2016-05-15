# Hands-On

Here we are going to experiment  how to communicate with a RGB LCD Display (http://www.seeedstudio.com/wiki/Grove_-_LCD_RGB_Backlight) and a barometric sensor (http://www.seeedstudio.com/wiki/Grove_-_Barometer_Sensor) from Grove, which by the way use I2C  as  communication protocol.

We will start with baby steps to later throw away UPM & MRAA:

* <font color="blue">Using libraries (UPM & MRAA)</font>
  * Shows how kids and not so hardcore nerds communicate with sensors
* <font color="blue">Look Ma! without Handles! (writing our own sensor code in user space)</font>
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

So far you should have found the following info about our sensor and display:

* Barometric Sensor: MPL3115A2
  * http://iotdk.intel.com/docs/master/upm/classupm_1_1_m_p_l3115_a2.html
* LCD Display: Jhd1313m2
  * http://iotdk.intel.com/docs/master/upm/classupm_1_1_jhd1313m1.html

####2. Writing some code with UPM/MRAA
Lets write a program that reads the temperature from the Barometric sensor and display it in our RGB LCD.

Go to your Edison's command-line and use **nano** or **vim** to create a file called <font color="blue">lcd_temp.cpp</font> and add the following code:

```
#include <unistd.h>
#include <iostream>
#include <signal.h>
#include <string>
#include <sstream>
#include "jhd1313m1.h"
#include "mpl3115a2.h"

volatile int doWork = 0;
void sig_handler(int signo)
{
    if (signo == SIGINT) {
        printf("\nCtrl-C received.\n");
        doWork = 1;
    }
}
int main(int argc, char **argv)
{
	signal(SIGINT, sig_handler);
	float pressure    = 0.0;
	float temperature = 0.0;
	float altitude    = 0.0;
	float sealevel    = 0.0;	
	 
 	upm::Jhd1313m1 *lcd = new upm::Jhd1313m1(0, 0x3E, 0x62);
 	upm::MPL3115A2 *barometer = new upm::MPL3115A2(0, MPL3115A2_I2C_ADDRESS);
	lcd->setCursor(0,0);
	lcd->setColor(127, 255, 127);
	lcd->write("Temperature:");
	std::stringstream ss;
    while (!doWork) 
    {

        temperature = barometer->getTemperature(true);
        pressure    = barometer->getPressure(false);
        altitude    = barometer->getAltitude();
        sealevel    = barometer->getSealevelPressure();
        ss.str(std::string());
        ss<<temperature;
        lcd->setCursor(1,2);
    	lcd->write(ss.str());
        usleep (500000);
    }

    delete lcd;
    delete barometer;
    return 0;     
}
```
now lets create a <font color="blue">Makefile</font> with **vim/nano** to compile easily compile the program every time we do a change.

Quick question!, can you identify the upm libs needed to compile our program?

```
Answer: lupm-i2clcd and lupm-mpl3115a2
   the name of the lib needed can be infered visually from the upm  page of the sensor (URLS at the beginning of this capter), take a look in the upper left corner ;)
```

Our Makefile should look like this:

```

all: lcd_temp

lcd_temp:
	g++ -lmraa -lupm-i2clcd -lupm-mpl3115a2 -I/usr/include/upm/ lcd_temp.cpp -o lcd_temp
clean:
	rm lcd_temp *.*~ .*~
```

now you can type **make**, to trigger the compilation of our program, after that type **./lcd_temp** to run the program, to stop it just press **CTRL+c** 

If you do some changes you can recompile using **make**, sometimes when you do some changes and recompile, it will tell you that there are no changes, on those occassions you can do **make clean** and then **make**  that will erase the tempfiles created by vim when editing as well as the executable binary.
want to learn more about the [Makefile](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html) system? click [here](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)

Using UPM/MRAA wasn't hard ins't it?, and it speed-up things if your sensor is already supported, but if not, you can write your own code and share it with he community, later  we will explore  how to add our not supported sensor to UPM.

####Look Ma! without Handles! 
In the earlier section we saw how to communicate witht he sensors using high level libraries, but imagine we are in the scenario where we are creating our own hardware and during the earlier stages of fine tuning our hardware it is very useful and easier to communicate using the linux low level tools and libraries :)  

* I2C Tools
  * i2cdetect
  * i2cdump
  * 12cset
  * i2cget


```
root@edison:~# i2cdetect -y -r 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- 48 -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- 62 -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: 70 -- -- -- -- -- -- -- 
```
```
i2cset -y 1 0x62 0x00 0x00
i2cset -y 1 0x62 0x08 0xFF
i2cset -y 1 0x62 0x01 0x20
i2cset -y 1 0x62 0x04 0xFF
i2cset -y 1 0x62 0x03 0xFF
i2cset -y 1 0x62 0x02 0xFF
```

####I'm a Pro!
