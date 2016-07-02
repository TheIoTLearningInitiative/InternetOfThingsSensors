# I2C

> The Inter-integrated Circuit (I2C) Protocol is a protocol intended to allow multiple “slave” digital integrated circuits (“chips”) to communicate with one or more “master” chips. Like the Serial Peripheral Interface (SPI), it is only intended for short distance communications within a single device. Like Asynchronous Serial Interfaces (such as RS-232 or UARTs), it only requires two signal wires to exchange information. [Sparkfun Learn about I2C](https://learn.sparkfun.com/tutorials/i2c)

> Creating, debugging and operating a custom I2C peripheral... [I2C Hacking Demystified](http://events.linuxfoundation.org/sites/events/files/slides/ELC%202016%20-%20I2C%20hacking%20demystified_0.pdf)

- [Linux Kernel I2C Documentation](https://www.kernel.org/doc/Documentation/i2c/)

# Hardware

- [Grove - LCD RGB Backlight (I2C Communication Protocol Device)](http://www.seeedstudio.com/wiki/Grove_-_LCD_RGB_Backlight)
- [PC9633 RGB controller](https://github.com/TheIoTLearningInitiative/InternetOfThingsSensors/blob/master/documentation/PCA9633.pdf)
    ![](http://www.seeedstudio.com/wiki/images/thumb/0/03/Serial_LEC_RGB_Backlight_Lcd.jpg/500px-Serial_LEC_RGB_Backlight_Lcd.jpg)

## Intel® Edison kit for Arduino expansion board.

![](http://shop-media.intel.com/api/v2/helperservice/getimage?url=http://images.icecat.biz/img/gallery/25435695_7315.jpg&height=550&width=550)

## SparkFun Block for Intel® Edison - I2C

> This I2C Block simply breaks out an I2C bus on the Intel® Edison while level shifting it from 1.8V to your sensors voltage. This a simple board that can snap into your Edison and be used right away. [SparkFun Block for Intel® Edison - I2C](https://www.sparkfun.com/products/13034)

![](https://cdn.sparkfun.com//assets/parts/1/0/0/3/5/13034-01_1.jpg)
# I2C Tools

> I2C device detection and register dump tools. These tools rely on the "i2c-dev" kernel driver.

You will need to get acquainted with a set of tools called I2C Tools, for our purposes the most common tools we are going to use are:

1. __i2cdetect__
2. __i2cdump__
3. __i2cget__
4. __i2cset__

## i2cdetect

It will help us list the i2c buses we have in our system and also show the addresses of the slaves connected to certain bus. Now go to your Edison and type <font color='blue'>i2cdetect -l</font> , you should get an output similar to this:

```sh
root@edison:~# i2cdetect -l
i2c-1	i2c       	i2c-designware-1                	I2C adapter
i2c-2	i2c       	i2c-designware-2                	I2C adapter
i2c-3	i2c       	i2c-designware-3                	I2C adapter
i2c-4	i2c       	i2c-designware-4                	I2C adapter
i2c-5	i2c       	i2c-designware-5                	I2C adapter
i2c-6	i2c       	i2c-designware-6                	I2C adapter
i2c-7	i2c       	i2c-designware-7                	I2C adapter
```

On the first Column we can see the **i2c-1** the leading number  indicates the number of the bus, in this case the bus is 1; 

Remember! **Edison uses bus 0 and 6 if you are using the Edison Arduino Breakout Board**,  but if you are using Edison  Breakout board or the **Sparkfun Base Block the Bus 0 will be 1** and here comes the tricky part, because when using Edison in the Arduino Breakout Board a module will be loaded,preventing us to see the slaves addresses in the bus, for example run this command: <font color='blue'>i2cdetect -y -r 1</font>  we will get:

### For Edison using Arduino Breakout Board

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

### For Edison using the Sparkfun Base Block

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

In the case of the Edison+ Sparkfun Base block i was able to see the i2c addresses from the LCD Display (where RGB controller is 0x62 and LCD controller is 0x70) and the ADC block (address 0x48). For this reason from now on we will use Edison with the [Sparkfun Base Block](https://www.sparkfun.com/products/13045) or the MiniBreakout Board, since they interface directly with the  Edison's I2C pins, but this doesn't mean it is not possible to use the arduino breakout board as we will see next, so keep reading!.


## i2cdump

It will show us the content of a specific register in our device, the syntax is:<font color='blue'>i2cdump -y BUS ADDRESS b</font> so go to your Edison and try to dump the registers from the RGB device of the Display. the command should be <font color='blue'>i2cdump -y 1 0x62 b</font>  and the result should look something like this:

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

**Answer:** the value for those 3 register is 0x00, to read the the dump we use the numbers from the left and the top like if they were coordinates, the sum of those values is equal to the register you want to read. i.e, taking the blue cirles we have  <font color='blue'>0x00</font> + <font color='blue'>0x04</font> = 0x04,  so the register 0x04 has the value <font color='red'>0x00</font> (red circle).

![](https://raw.githubusercontent.com/TheIoTLearningInitiative/InternetOfThingsSensors/master/images/dump.png)

Now for the question about what these registers represent, if you read datasheet you will find that they are PWM registers 0 to 3, PWMx — Individual brightness control registers these registers, and here it involves another protocol PWM which stands for Pulse width modulation (go check the protocols section). As color is a mix of Red, Green and Blue we can mix light by adjusting its brightnes and this registers do just that the control bright in a scale of 0 to 255 :) 

To see the other available registers take a look to page 11 of the [PC9633 RGB controller](https://github.com/TheIoTLearningInitiative/InternetOfThingsSensors/blob/master/documentation/PCA9633.pdf) that has the Grove display.

## i2cset & i2cget

These guys are used to set or retrieve the value of a specific register in our i2c device, so lets go and initiate the RGB controller of our Display and set the color to white :), you already know to what registers we should write (you found them in your datasheet) to set the color.

<font color='blue'></font>

Well in your Edison's command-line type each command and hit enter :)

### For Edison using Arduino Breakout Board

```
root@edison:~# i2cset -y 6 0x62 0x00 0x00
root@edison:~# i2cset -y 6 0x62 0x08 0xFF
root@edison:~# i2cset -y 6 0x62 0x01 0x20
root@edison:~# i2cset -y 6 0x62 0x04 0xFF
root@edison:~# i2cset -y 6 0x62 0x03 0xFF
root@edison:~# i2cset -y 6 0x62 0x02 0xFF
```

### For Edison using the Sparkfun Base Block

```
root@edison:~# i2cset -y 6 0x62 0x00 0x00
root@edison:~# i2cset -y 6 0x62 0x08 0xFF
root@edison:~# i2cset -y 6 0x62 0x01 0x20
root@edison:~# i2cset -y 6 0x62 0x04 0xFF
root@edison:~# i2cset -y 6 0x62 0x03 0xFF
root@edison:~# i2cset -y 6 0x62 0x02 0xFF
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
See?, 0x04, 0x03, 0x02 now have 0xFF as their value, try setting the color to a solid **<font color="green">Green</font>**


---


**Answer**: you don't need to re initiate the RGB controller, just set the RGB registers to the correct values, to set the color to Green, we need R=0, G=255, B=0. so we only need to do:

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

Now from the list in my Edison I can see i have the option to install **i2c-tools-dev - 3.1.1-r0** if it doesn't show in your Edison then you will have to configure some repositories to have access to them since the i2c-dev libraries are not available by default, so use **vim/nano** to review **/etc/opkg/base-feeds.conf** where you should have something like this:

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
