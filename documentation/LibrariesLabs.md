# Labs

# Using libraries (UPM & MRAA)

Do not misunderstand me , UPM and MRAA are great libraries, they really make things easier, specially when you are starting and you would like to get a prototype up and running fast, instead of struggling in how to get the values from your sensors and devices.

So first let's recap. MRAA is used to communicate to the device through I2C, PWM,SPI, etc.. on top of that UPM is used so at the end you just have to instantiate an object of the class of your device or sensor. lets see how we can do that:

## 1. Check if the sensors are supported by MRAA

At the beginning we stated that we are going use the LCD RGB  Backlight Display and the Barometric sensor, please go to http://iotdk.intel.com/docs/master/upm/modules.html and find out either using the search on the top right corner or manually browse the sensor list to see if our sensors are supported by MRAA. 

So far you should have found the following info about our sensor and display:

* Barometric Sensor: MPL3115A2
  * http://iotdk.intel.com/docs/master/upm/classupm_1_1_m_p_l3115_a2.html
* LCD Display: Jhd1313m2
  * http://iotdk.intel.com/docs/master/upm/classupm_1_1_jhd1313m1.html

## 2. Writing some code with UPM/MRAA

Lets write a program that reads the temperature from the Barometric sensor and display it in our RGB LCD.

Go to your Edison's command-line and use **nano** or **vim** to create a file called <font color="blue">lcd_temp.cpp</font> and add the following code:

```sh
lcd_temp.cpp
```

```c
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
	rm lcd_temp
```

now you can type **make**, to trigger the compilation of our program, after that type **./lcd_temp** to run the program, to stop it just press **CTRL+c** 

If you do some changes you can recompile using **make**, sometimes when you do some changes and recompile, it will tell you that there are no changes, on those occassions you can do **make clean** and then **make**  that will erase the tempfiles created by vim when editing as well as the executable binary.
want to learn more about the [Makefile](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html) system? click [here](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)

Using UPM/MRAA wasn't hard ins't it?, and it speed-up things if your sensor is already supported, but if not, you can write your own code and share it with he community, later  we will explore  how to add our not supported sensor to UPM.

