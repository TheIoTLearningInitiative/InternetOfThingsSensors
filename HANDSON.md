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
	rm lcd_temp
```

now you can type **make**, to trigger the compilation of our program, after that type **./lcd_temp** to run the program, to stop it just press **CTRL+c** 

If you do some changes you can recompile using **make**, sometimes when you do some changes and recompile, it will tell you that there are no changes, on those occassions you can do **make clean** and then **make**  that will erase the tempfiles created by vim when editing as well as the executable binary.
want to learn more about the [Makefile](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html) system? click [here](https://www3.ntu.edu.sg/home/ehchua/programming/cpp/gcc_make.html)

Using UPM/MRAA wasn't hard ins't it?, and it speed-up things if your sensor is already supported, but if not, you can write your own code and share it with he community, later  we will explore  how to add our not supported sensor to UPM.

####Look Ma! without Handles! 
In the earlier section we saw how to communicate with the sensors using high level libraries, but imagine we are in the scenario where we are creating our own hardware, during the earlier stages of fine tuning our hardware it is very useful and easier to communicate using the linux low level tools and libraries :), libraries that even MRAA and almost all linux drivers relies on. 

You will need to get acquainted with a set of tools called I2C Tools, for our purpouses the most common tools we are going to use are:

1. ***i2cdetect***
2. ***i2cdump***
3. ***i2cget***
4. ***i2cset***


**i2cdetect** : will help us list the i2c buses we have in our system and also show the addresses of the slaves connected to certain bus. Now go to your Edison and type <font color='blue'>i2cdetect -l</font> , you should get an output similar to this:

```
root@edison:~# i2cdetect -l
i2c-1	i2c       	i2c-designware-1                	I2C adapter
i2c-2	i2c       	i2c-designware-2                	I2C adapter
i2c-3	i2c       	i2c-designware-3                	I2C adapter
i2c-4	i2c       	i2c-designware-4                	I2C adapter
i2c-5	i2c       	i2c-designware-5                	I2C adapter
i2c-6	i2c       	i2c-designware-6                	I2C adapter
i2c-7	i2c       	i2c-designware-7                	I2C adapter

```
On the first Column we can see the **i2c-1** the leading number  indicates the number of the bus, in this case the bus is 1; Remember **Edison uses bus 0 and 6 if you are using the Edison Arduino Breakout Board**,  but if you are using Edison  Breakout board or the **Sparkfun Base Block the Bus 0 will be 1** and here comes the tricky part, because when using Edison in the Arduino Breakout Board a module will be loaded,preventing us to see the slaves addresses in the bus, for example run this command: <font color='blue'>i2cdetect -y -r 1</font>  we will get:

**For Edison using Arduino Breakout Board**
```
root@edison:~# i2cdetect -y -r 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: UU UU UU UU -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --  
```
**For Edison using the Sparkfun Base Block**
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
For this reason from now on we will use Edison with the [Sparkfun Base Block](https://www.sparkfun.com/products/13045) or the MiniBreakout Board, since they interface directly with the  Edison's I2C pins, but this doesn't mean it is not possible to use the arduino breakout board as we will see next, so keep reading!.


**i2cdump**: will show us the content of a specific register in our device, the syntax is:<font color='blue'>i2cdump -y BUS ADDRESS b</font> so go to your edison and try to dumo the registers from the RGB device of the Display. the command should be <font color='blue'>i2cdump -y 1 0x62 b</font>  and the result should look something like this:

```
root@edison:~# i2cdump -y  1 0x62 b
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: 11 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
10: 11 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
20: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
30: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
40: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
50: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
60: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
70: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
80: 91 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
90: 91 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
a0: b1 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
b0: b1 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
c0: d1 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
d0: d1 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
e0: f1 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
f0: f1 01 00 00 00 00 ff 00 00 e2 e4 e8 e0 XX XX XX    ??.......????XXX
```

Please from the above dump, can you determinate what is the value of register 0x04, 0x03, 0x02?, do you know to what those registers correspond to?, check the datasheet for the RGB controller.


---


**Answer:** the value for those 3 register is 0x00, to read the the dump we use the numbers from the left and the top like if they were coordinates, the sum of hose value is equal to the register you want to read. i.e, taking the blue cirles we have  <font color='blue'>0x00</font> + <font color='blue'>0x04</font> = 0x04,  so the register 0x04 has the value <font color='red'>0x00</font> (red circle).

![](images/dump.png)

now for the question about what these registers represent, if you read datasheet you will find that they are PWM registers 0 to 3, PWMx — Individual brightness control registers these registers, and here it involves another protocol PWM which stands for Pulse width modulation (go check the protocols section). As color is a mix of Red, Green and Blue we can mix light by adjusting its brightnes and this registers do just that the control bright in a scale of 0 to 255 :) 

To see the other available registers take a look to page 11 of the [PC9633](documentation/PCA9633.pdf) RGB controller that has the Grove display.


---



**12cset & i2cget**: these guys are used to set or retrieve the value of a specific register in our i2c device, so lets go and initiate the RGB controller of our Display and set the color to white :), you already know to what registers we should write (you found them in your datasheet) to set the color.

<font color='blue'></font>

Well in your Edison's command-line type each command and hit enter :)
```
i2cset -y 1 0x62 0x00 0x00
i2cset -y 1 0x62 0x08 0xFF
i2cset -y 1 0x62 0x01 0x20
i2cset -y 1 0x62 0x04 0xFF
i2cset -y 1 0x62 0x03 0xFF
i2cset -y 1 0x62 0x02 0xFF
```
What about that!, you just tapped into your i2c device, init the RGB controller and set the color to white, without any program!...

now take a look to lets say the register in charge of setting the red color(0x04) using **i2cget**. type this:<font color='blue'> i2cget -y 1 0x62 0x04</font>.

You should see this:
```
root@edison:~# i2cget -y 1 0x62 0x04
0xff
```
So, 0xFF (or 255 in decimal) is just the value we set using i2cset :) you can also check that by dumping the RGB controller and you should see something like this:

```
root@edison:~# i2cdump -y  1 0x62 b
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: 00 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    . .......????XXX
10: 00 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    . .......????XXX
20: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
30: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
40: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
50: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
60: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
70: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
80: 80 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX
90: 80 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX
a0: a0 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX
b0: a0 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX
c0: c0 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX
d0: c0 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX
e0: e0 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX
f0: e0 20 ff ff ff 00 ff 00 ff e2 e4 e8 e0 XX XX XX    ? .......????XXX

```
See?, 0x04, 0x03, 0x02 no have 0xFF as their values, try setting the color to a solid **<font color="green">Green</font>**


---


**Answer**: you don't need to re initiate th RGB controller, just set the RGB registers to the correct values, to set the color to Green, we need R=0, G=255, B=0. so we only need to do:

```
i2cset -y 1 0x62 0x04 0x00
i2cset -y 1 0x62 0x03 0xFF
i2cset -y 1 0x62 0x02 0x00
```

and BUM!... Yoda green it is!...

---


Remember i told you it is not quite impossible to do this on the Edison + Arduino Breakout Board?; well before i tell youhow we can do a workaround  lets remember.. what are the lines needed by a device so I2C protocol can do its job?

* Vdd (power)
* GND (ground)
* SDA (Data line)
* SCL (Clock line) 

Now go and check the [Intel® Edison Kit for Arduino* Hardware Guide
](http://www.intel.com/content/www/us/en/support/boards-and-kits/000005583.html) page 7, we know that the Arduino SDA and SCL pins are connected to bus 1, and whne you connect a device it doesn't show when you try to do an i2cdetect so what are alternatives do we have?


---


**Answer**: There are other pins that offer the SDA and SCL and those correspond to Bus 6  and these pins are the Arduino's A4 and A5 in our Arduino Break-out Board!.. so basically we can connect the cables from the LCD like this:

| Arduino Pin | LCD I2C pin|
| -- | -- |
|5v | Vdd |
|GND | GND |
|A4 | SDA |
|A5 | SCL |


---

Talking to our I2C devices using I2CTools is cool, and makes you look interesting, but lets face it  it is not practical enough, and option can be put everything in a shell script but we want to get further than that like writing our own module and for that  lets start creating our grown-ups c program and for that i really reccommend reading the [I2C Documentation in linux](https://www.kernel.org/doc/Documentation/i2c/) (we'll be taking base code snippet from there) and about [i2c-dev library](http://lxr.free-electrons.com/source/drivers/i2c/i2c-dev.c) we will be usingit to develope our C program.

First lets see if we have what we need in our Edison and equip it  if not. So type <font color="blue">opkg list|grep i2c</font> and lets findout if you have something similar to this:

```
root@edison:~# opkg list |grep i2c
i2c-tools - 3.1.1-r0 - Set of i2c tools for linux  Set of i2c tools for linux.
i2c-tools-dbg - 3.1.1-r0 - Set of i2c tools for linux - Debugging files  Set of i2c tools for linux.
i2c-tools-dev - 3.1.1-r0 - Set of i2c tools for linux - Development files  Set of i2c tools for
i2c-tools-doc - 3.1.1-r0 - Set of i2c tools for linux - Documentation files  Set of i2c tools for
i2c-tools-misc - 3.1.1-r0 - Set of i2c tools for linux  Set of i2c tools for linux.
python-smbus - 3.1.1-r0 - Set of i2c tools for linux - Python module  Set of i2c tools for linux -
python-smbus-dbg - 3.1.1-r0 - Set of i2c tools for linux - Python module - Debugging files  Set of i2c
python-smbus-dev - 3.1.1-r0 - Set of i2c tools for linux - Python module - Development files  Set of
 i2c tools for linux - Python module.  This package contains   symbolic

```

Now from the list in my Edison i cna see i have the option to install **i2c-tools-dev - 3.1.1-r0** if it doesn't show in your Edison then you will have to confgure some repositories to have access to them since the i2c-dev libraries are not available by default, so use **vim/nano** to review **/etc/opkg/base-feeds.conf** where you should have something like this:

```
src/gz all http://repo.opkg.net/edison/repo/all
src/gz edison http://repo.opkg.net/edison/repo/edison
src/gz core2-32 http://repo.opkg.net/edison/repo/core2-32
```
if you don't have it please add those lines to your file, for further help or instructions you can visit [AlexT's Galileo & Edison pages](http://alextgalileo.altervista.org/edison-package-repo-configuration-instructions.html)

Once you have done editing, save and close the file and run **opkg update** to start downloading the list of available packages from those repos. After that  check again with <font color="blue">opkg list | grep i2c</font> and verify you have the option to install **i2c-tools-dev - 3.1.1-r0**

For those that already added the repo and want to check if it is already installed you can type <font color="blue">opkg list-installed | grep i2c</font> and should see something like this:
```
root@edison:~# opkg list-installed | grep i2c
i2c-tools - 3.1.1-r0
i2c-tools-dev - 3.1.1-r0
i2c-tools-misc - 3.1.1-r0
```
if not, to install it run <font color="blue">opkg install i2c-tools-dev </font> and we are done!


Now lets start by creating a file called **lcd.c**  and start  adding the headers we're going to need and the body of the program:

```
#include <errno.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <linux/i2c-dev-user.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdbool.h>


int main()
{

	return 0;
}
```


now lets do some defines so we can have the registers we need at hand, lets start with those registers needed by the RGB controller of our Display.

```
#define RGB_SLAVE       0x62
#define LCD_SLAVE	   0x3E
#define BUS		     0x01
#define REG_RED         0x04        // pwm2
#define REG_GREEN       0x03        // pwm1
#define REG_BLUE        0x02        // pwm0
#define REG_MODE1       0x00
#define REG_MODE2       0x01
#define REG_OUTPUT      0x08
```

As an practice excercise, please go to the datasheet and determinate what we are doing with those values, are those values addresses or are they configuration settings?



---
**Answer**: they are register addresses, as depicted in page 11 of the PCA9633 Datasheet (that's our RGB Display controller). It is a good idea to try understand how this registers work together.

---




According to the linux documentation we have to open a device file in order to communicate with our device so this can act as a i2c context for any future communication so lets create a struct that can store all that information named I2CCONTEXT.

```
typedef struct
{
    int addr; //i2c address of device
    int file; //reprsents a place to do the i2c read & write
    int adapter_nr; //the bus number of the device
    char filename[20]; //the name of the device
}I2CCONTEXT;
I2CCONTEXT lcd; //will store the lcd controller context
I2CCONTEXT rgb; //will store the rgb controller context
```

Now lets create a function where we are going to create the context, this function will receive the reference to an **I2CCONTEXT** variable, the **address** of the device and the **bus** where it is laying. as a return value we can send the errno value:
```
int initContext(I2CCONTEXT *ctx, int addr_,int bus)
{
	ctx->addr = addr_;
	ctx->adapter_nr = bus;
	snprintf(ctx->filename, 19, "/dev/i2c-%d", ctx->adapter_nr);
	ctx->file = open(ctx->filename, O_RDWR);
    
	if (ctx->file < 0) 
	{
	   /* ERROR HANDLING; you can check errno 2 c what went wrong */
		printf("Error ocurred @ opening BUS: (errno:%d) %s\n",
				errno,
				strerror(errno));
		return -errno;	

	}
	
	if (ioctl(ctx->file, I2C_SLAVE, ctx->addr) < 0)
	{
        /* ERROR HANDLING; you can check errno 2 c what went wrong */
		printf("Error ocurred @ accessing slave: (errno:%d) %s\n",
					errno,
					strerror(errno));
		return -errno;
	}

}
```

now lets add some  methods to do the reading and writing of the i2c registers, basically there are two flavors, one is using smbus or pure i2c methods, we will use smbus  since it is supported in Edison, and is a bit easier since there are already  methods that handle register lenghts of byte and word, otherwise we will have to create a buffer and fill it according hte i2c protocol,  it is left as an excercise for the reader to use i2c pure methods.

Alright! so if you inspect the datasheets ( go figure! :P) you will notice that the length of the registers is 1 byte (8 bits) so from the i2c-dev methods  there one that seems to be just what we need to write byte data: 

```
i2c_smbus_write_byte_data(int file, __u8 command, __u8 value);
```
and to read a register we can use:

```
i2c_smbus_read_word_data(int file, __u8 register, __u8 value);
```
lets create two methods so we can handle any errors and return the error code in case something goes wrong.

```
__s32 writeByteRegister(int file, __u8 register_, __u8 value)
{
	__s32 res = -1;
	res = i2c_smbus_write_byte_data(file, register_, value);
	if (res < 0)
	{
		/* ERROR HANDLING: i2c transaction failed */
		printf("Error writing byte, (errno:%d),%s",errno,
				strerror(errno));
	}
}

__s32 readRegister(int register_, int file)
{
	__u8 reg = register_;
	__s32 res = -1;
	char buf[10];  
	res = i2c_smbus_read_word_data(file, reg);
	if (res < 0)
	{
	    /* ERROR HANDLING: i2c transaction failed */
		printf("Error reading reg: 0x%x, (errno:%d),%s",
				reg,errno,strerror(errno));
	}
	return res;
}
```
Now lets  test if we can get the I2C context of our RGB controller for that, add this to the  main():

```
/*
we pass a reference to the rgb context variable
the i2c address of the rgb controller
and the BUS which should be 1
*/
initContext(&rgb, RGB_SLAVE , BUS);
printf("\nDONE!\n");
```


so far your **lcd.c** sourcecode should look like this:

```
#include <errno.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <linux/i2c-dev-user.h>
#include <sys/ioctl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdbool.h>

#define RGB_SLAVE       0x62
#define LCD_SLAVE       0x3E
#define BUS             0x01
#define REG_RED         0x04        // pwm2
#define REG_GREEN       0x03        // pwm1
#define REG_BLUE        0x02        // pwm0
#define REG_MODE1       0x00
#define REG_MODE2       0x01
#define REG_OUTPUT      0x08

typedef struct
{
    int addr; //i2c address of device
    int file; //reprsents a place to do the i2c read & write
    int adapter_nr; //the bus number of the device
    char filename[20]; //the name of the device
}I2CCONTEXT;

I2CCONTEXT lcd; //will store the lcd controller context
I2CCONTEXT rgb; //will store the rgb controller context

int initContext(I2CCONTEXT *ctx, int addr_,int bus)
{
    ctx->addr = addr_;
    ctx->adapter_nr = bus;
    snprintf(ctx->filename, 19, "/dev/i2c-%d", ctx->adapter_nr);
    ctx->file = open(ctx->filename, O_RDWR);

    if (ctx->file < 0) 
    {
       /* ERROR HANDLING; you can check errno 2 c what went wrong */
        printf("Error ocurred @ opening BUS: (errno:%d) %s\n",
                errno,
                strerror(errno));
        return -errno;    

    }

    if (ioctl(ctx->file, I2C_SLAVE, ctx->addr) < 0)
    {
        /* ERROR HANDLING; you can check errno 2 c what went wrong */
        printf("Error ocurred @ accessing slave: (errno:%d) %s\n",
                    errno,
                    strerror(errno));
        return -errno;
    }

}

__s32 writeByteRegister(int file, __u8 register_, __u8 value)
{
    __s32 res = -1;
    res = i2c_smbus_write_byte_data(file, register_, value);
    if (res < 0)
    {
        /* ERROR HANDLING: i2c transaction failed */
        printf("Error writing byte, (errno:%d),%s",errno,
                strerror(errno));
    }
}

__s32 readRegister(int register_, int file)
{
    __u8 reg = register_;
    __s32 res = -1;
    char buf[10];  
    res = i2c_smbus_read_word_data(file, reg);
    if (res < 0)
    {
        /* ERROR HANDLING: i2c transaction failed */
        printf("Error reading reg: 0x%x, (errno:%d),%s",
                reg,errno,strerror(errno));
    }
    return res;
}

int main()
{

    /*
	we pass a reference to the rgb context variable
	the i2c address of the rgb controller
	and the BUS which should be 1
    */
    initContext(&rgb, RGB_SLAVE , BUS);
    printf("\nDONE!\n");
    return 0;
}
```
now lets create a **Makefile**  with the following content:

```
all:lcdtest

lcdtest:lcd.c
		gcc -O lcd.c -o lcdtest
clean:
		@rm lcdtest
```
run **make** and when the compilation succesfully finishes run **./lcdtest**

Once your code is working lets add a method to init our RGB controller, also we will need a method to set the color of the Display :

```
void setRGBColor(I2CCONTEXT *rgb, int r, int g, int b)
{
    	writeByteRegister(rgb->file, REG_RED, r);
        writeByteRegister(rgb->file, REG_GREEN, g);
    	writeByteRegister(rgb->file, REG_BLUE, b);            
}
void initRGB(I2CCONTEXT *rgb)
{
    	// backlight init
    	writeByteRegister(rgb->file, REG_MODE1, 0);
    	// set LEDs controllable by both PWM and GRPPWM registers
    	writeByteRegister(rgb->file, REG_OUTPUT, 0xFF);
    	// set MODE2 values
        writeByteRegister(rgb->file, REG_MODE2, 0x20);
        // set the baklight Color to white :)
        setRGBColor(rgb, 0xFF, 0xFF, 0xFF);	
}
```
add another method to turn off the light upon program exit:

```
void turnOffRGB(I2CCONTEXT *rgb)
{
	setRGBColor(rgb, 0x00, 0x00, 0x00);	
}
```
now lets call init RGB from our main() by passing the reference to our RGB context waith about 5 seconds and then exit turning off the color leds:
```
/*sleep for 5 secs before turning off*/
sleep(5);
/*turn off RGB LEDS*/
turnOffRGB(&rgb);
```





run **make clean** then **make** and run by typing **./lcdtest**
####I'm a Pro!
