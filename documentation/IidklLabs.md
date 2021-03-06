# Laboratory

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

## 2. Writing some code with UPM/MRAA, LCD

Let's write a program to display a string in our RGB LCD.

Go to your Edison's command-line and use **nano** or **vim** to create a file called <font color="blue">lcd_temp.cpp</font> and add the following code:

```sh
root@edison:~# cd TheIoTLearningInitiative/Sensors/IoTDKL
root@edison:~/TheIoTLearningInitiative/Sensors/IoTDKL# 
```

```sh
root@edison:~/.../IoTDKL# nano lcd.cpp
```

```c
#include <unistd.h>                                                             
#include <iostream>                                                             
#include <signal.h>                                                             
#include <string>                                                               
#include <sstream>                                                              
#include "jhd1313m1.hpp"                                                        
                                                                                
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
                                                                                
        upm::Jhd1313m1 *lcd = new upm::Jhd1313m1(0, 0x3E, 0x62);                
        lcd->setCursor(0,0);                                                    
        lcd->setColor(127, 255, 127);                                           
        lcd->write("Hello LCD");
        while (!doWork)                                                         
        {                                                                       
                usleep (500000);                                                
        }                                                                       
                                                                                
        delete lcd;                                                             
        return 0;                                                               
}
```

## 3. Compiling our UPM/MRAA code with a Makefile

Now lets create a <font color="blue">Makefile</font> with **vim/nano** to compile easily compile the program every time we do a change.

Quick question!, can you identify the upm libs needed to compile our program?

```
Answer: lupm-i2clcd and lupm-mpl3115a2
   the name of the lib needed can be infered visually from the upm  page of the sensor (URLS at the beginning of this capter), take a look in the upper left corner ;)
```

Our Makefile should look like this:

```sh
root@edison:~/.../IoTDKL# nano Makefile
```

```c
all: lcd

lcd:
	g++ -lmraa -lupm-i2clcd -I/usr/include/upm/ lcd.cpp -o lcd
clean:
	rm lcd
```

Now you can type **make**, to trigger the compilation of our program, after that type **./lcd** to run the program, to stop it just press **CTRL+c** 

```sh
root@edison:~/.../IoTDKL# make
g++ -lmraa -lupm-i2clcd -lupm-mpl3115a2 -I/usr/include/upm/ lcd.cpp -o lcd
root@edison:~/.../IoTDKL#  
```

```sh
root@edison:~/.../IoTDKL# ./lcd 
^C
Ctrl-C received.
root@edison:~/.../IoTDKL#   
```

If you do some changes you can recompile using **make**, sometimes when you do some changes and recompile, it will tell you that there are no changes, on those occassions you can do **make clean** and then **make**  that will erase the tempfiles created by vim when editing as well as the executable binary.

```sh
root@edison:~/.../IoTDKL# make clean
rm lcd                                                                
root@edison:~/.../IoTDKL#   
```

## 4. Writing some code with UPM/MRAA, Temperature

Now lets read the temperature from the Barometric sensor and display it in our RGB LCD.

Go to your Edison's command-line and use **nano** or **vim** to create a file called <font color="blue">lcd_temp.cpp</font> and add the following code:

```sh
root@edison:~/.../IoTDKL# nano lcd_temp.cpp
```

```c
#include <unistd.h>
#include <iostream>
#include <signal.h>
#include <string>
#include <sstream>
#include "jhd1313m1.hpp"
#include "mpl3115a2.hpp"

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

Our Makefile should look like this:

```sh
root@edison:~/.../IoTDKL# nano Makefile
```

```c
all: lcd lcd_temp

lcd:
	g++ -lmraa -lupm-i2clcd -lupm-mpl3115a2 -I/usr/include/upm/ lcd.cpp -o lcd
lcd_temp:
	g++ -lmraa -lupm-i2clcd -lupm-mpl3115a2 -I/usr/include/upm/ lcd_temp.cpp -o lcd_temp
clean:
	rm lcd lcd_temp
```

Now compile

```sh
root@edison:~/.../IoTDKL# make
g++ -lmraa -lupm-i2clcd -lupm-mpl3115a2 -I/usr/include/upm/ lcd.cpp -o lcd
g++ -lmraa -lupm-i2clcd -lupm-mpl3115a2 -I/usr/include/upm/ lcd_temp.cpp -o lcd_temp
root@edison:~/.../IoTDKL#   
```

```sh
root@edison:~/.../IoTDKL# ./lcd_temp 
^C
Ctrl-C received.
root@edison:~/.../IoTDKL#   
```

```sh
root@edison:~/.../IoTDKL# cd
root@edison:~# 
```

Want to learn more about the [Makefile](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html) system? click [Here: GCC and Make Compiling, Linking and Building C/C++ Applications](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)

Using UPM/MRAA wasn't hard ins't it?, and it speed-up things if your sensor is already supported, but if not, you can write your own code and share it with the community, later  we will explore  how to add our not supported sensor to UPM.

