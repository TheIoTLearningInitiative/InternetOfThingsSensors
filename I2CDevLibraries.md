# I2C Dev Libraries

# Look Ma! without Handles!

In the earlier section we saw how to communicate with the sensors using high level libraries, but imagine we are in the scenario where we are creating our own hardware, during the earlier stages of fine tuning our hardware it is very useful and easier to communicate using the linux low level tools and libraries :), libraries that even MRAA and almost all Linux drivers relies on. 

Now lets start by creating a file called **lcd.c**  and start  adding the headers we're going to need and the body of the program:

```sh
root@edison:~# cd TheIoTLearningInitiative/Sensors
root@edison:~/TheIoTLearningInitiative/Sensors# 
```

```sh
root@edison:~/TheIoTLearningInitiative/Sensors# mkdir I2CDL
root@edison:~/TheIoTLearningInitiative/Sensors# cd I2CDL/
root@edison:~/TheIoTLearningInitiative/Sensors/I2CDL# 
```

```sh
root@edison:~/.../I2CDL# nano lcd.c
```

```c
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
#include <inttypes.h>


int main()
{

	return 0;
}
```

Now lets do some defines so we can have the registers we need at hand, lets start with those registers needed by the RGB controller of our Display.

```c
#define RGB_SLAVE       0x62
#define LCD_SLAVE       0x3E
#define BUS             0x01
#define REG_RED         0x04        // PWM2
#define REG_GREEN       0x03        // PWM1
#define REG_BLUE        0x02        // PWM0
#define REG_MODE1       0x00
#define REG_MODE2       0x01
#define REG_OUTPUT      0x08
```

As an practice excercise, please go to the [PC9633 RGB Controller](https://github.com/TheIoTLearningInitiative/InternetOfThingsSensors/blob/master/documentation/PCA9633.pdf) and determinate what we are doing with those values, are those values addresses or are they configuration settings?

---

**Answer**: they are register addresses, as depicted in page 11 of the [PC9633 RGB Controller](https://github.com/TheIoTLearningInitiative/InternetOfThingsSensors/blob/master/documentation/PCA9633.pdf) (that's our RGB Display controller). It is a good idea to try understand how this registers work together.

---

According to the linux documentation we have to open a device file in order to communicate with our device so this can act as a i2c context for any future communication so lets create a struct that can store all that information named I2CCONTEXT.

```c
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

Now lets create a function where we are going to create the context, this function will receive the reference to an **I2CCONTEXT** variable, the **address** of the device and the **bus** where it is laying. as a return value we can send the errno value:ç

```c
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

Now lets add some  methods to do the reading and writing of the i2c registers, basically there are two flavors, one is using smbus or pure i2c methods, we will use smbus  since it is supported in Edison, and is a bit easier since there are already  methods that handle register lenghts of byte and word, otherwise we will have to create a buffer and fill it according hte i2c protocol,  it is left as an excercise for the reader to use i2c pure methods.

Alright! so if you inspect the datasheets ( go figure! :P) you will notice that the length of the registers is 1 byte (8 bits) so from the i2c-dev methods  there one that seems to be just what we need to write byte data: 

```c
i2c_smbus_write_byte_data(int file, __u8 command, __u8 value);
```

and to read a register we can use:

```c
i2c_smbus_read_word_data(int file, __u8 register, __u8 value);
```

lets create two methods so we can handle any errors and return the error code in case something goes wrong.

```c
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

```c
/*
we pass a reference to the rgb context variable
the i2c address of the rgb controller
and the BUS which should be 1
*/
initContext(&rgb, RGB_SLAVE , BUS);
printf("\nDONE!\n");
```


so far your **lcd.c** sourcecode should look like this:

```sh
root@edison:~/.../I2CDL# nano lcd.c
```

```c
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

Now lets create a **Makefile**  with the following content:

```sh
root@edison:~/.../I2CDL# nano Makefile
```

```
all: lcdtest

lcdtest:lcd.c
		gcc -O lcd.c -o lcdtest
clean:
		@rm lcdtest
```

Run **make** and when the compilation succesfully finishes run **./lcdtest**

```sh
root@edison:~/.../I2CDL# make
gcc -O lcd.c -o lcdtest
```

```sh
root@edison:~/.../I2CDL# 
                                                                                           
DONE! 
```


Once your code is working lets add a method to init our RGB controller, also we will need a method to set the color of the Display :

```c
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

Add another method to turn off the light upon program exit:

```c
void turnOffRGB(I2CCONTEXT *rgb)
{
	setRGBColor(rgb, 0x00, 0x00, 0x00);	
}
```

now lets call init RGB from our main() by passing the reference to our RGB context wait about 5 seconds and then exit turning off the color leds:
```
/*turn on RGB LEDS*/
initRGB(&rgb);
/*sleep for 5 secs before turning off*/
sleep(5);
/*turn off RGB LEDS*/
turnOffRGB(&rgb);
```

run **make clean** then **make** and execute by typing **./lcdtest**

```sh
root@edison:~/.../I2CDL# make
gcc -O lcd.c -o lcdtest
```

```sh
root@edison:~/.../I2CDL# 
                                                                                           
DONE! 
```

Now lets write some methods to drive the LCD controller, first of all the JHD1313 datasheet (Grove for some reason puts a link to the datasheet of JHD1214) is not too clear, but after some tome you will realize that to init the device you can write to register 0x00 if you want to write to CGRAM AD you need to write to register 0x40 and to write information to DDRAM you need to write to register 0x80 (take a look to the [JHD1214 data sheet](http://www.seeedstudio.com/wiki/images/0/03/JHD1214Y_YG_1.0.pdf)).

So lets start adding some defines we will need:

```
// commands
#define LCD_CLEARDISPLAY	0x01
#define LCD_RETURNHOME	  0x02
#define LCD_ENTRYMODESET	0x04
#define LCD_DISPLAYCONTROL	0x08
#define LCD_CURSORSHIFT	0x10
#define LCD_FUNCTIONSET 0x20
#define LCD_SETCGRAMADDR	0x40
#define LCD_SETDDRAMADDR	0x80

// flags for display entry mode
#define LCD_ENTRYRIGHT	0x00
#define LCD_ENTRYLEFT	0x02
#define LCD_ENTRYSHIFTINCREMENT 0x01
#define LCD_ENTRYSHIFTDECREMENT 0x00

// flags for display on/off control
#define LCD_DISPLAYON	0x04
#define LCD_DISPLAYOFF	0x00
#define LCD_CURSORON	0x02
#define LCD_CURSOROFF	0x00
#define LCD_BLINKON	0x01
#define LCD_BLINKOFF	0x00

// flags for display/cursor shift
#define LCD_DISPLAYMOVE 0x08
#define LCD_CURSORMOVE	0x00
#define LCD_MOVERIGHT	0x04
#define LCD_MOVELEFT	0x00

// flags for function set
#define LCD_8BITMODE	0x10
#define LCD_4BITMODE	0x00
#define LCD_2LINE	0x08
#define LCD_5x8DOTS	0x00
//////////////////////////////////////////////////////////
uint8_t _displayfunction = 0;
uint8_t _displaycontrol = 0;
uint8_t _displaymode = 0;
uint8_t lines = 2;
```

Now add an init method to setup our LCD:

```
void initLCD(I2CCONTEXT *lcd)
{

    //set the number of lines of our Display
    ////////////////////////////////////////////////////////////////////
	writeByteRegister(lcd->file, 0x00, LCD_FUNCTIONSET | LCD_2LINE );
	usleep(100);
    
    //enable display, show cursor, show a blinking box cursor
    ////////////////////////////////////////////////////////////////////
	writeByteRegister(lcd->file, 0x00,LCD_DISPLAYCONTROL | LCD_DISPLAYON | LCD_CURSORON | LCD_BLINKON );
	usleep(100);
    
    //clear display and set cursor at 0 position
    ////////////////////////////////////////////////////////////////////
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );
	usleep(4000);
    
    //set how the cursor will move everytime a letter is written to the LCD
    ////////////////////////////////////////////////////////////////////////
	writeByteRegister(lcd->file, 0x00,LCD_ENTRYMODESET | LCD_ENTRYLEFT | LCD_ENTRYSHIFTDECREMENT );

}
```

Of course we need a function we can pass a c string to it , then iterate it  and pass each letter to the LCD. for that lets add the following method:

```
void writeToLCD(I2CCONTEXT *lcd, char *c)
{

	uint ascii_val=0;
	int i;
	int s = strlen(c);
	for(i = 0 ; i < s; i++)
	{
		ascii_val = toascii(c[i]);
        ///////////////////////////////////////////////////////////
        //sending a 0x40 tell the LCD we will send a character code
        //to CGRAM
        ///////////////////////////////////////////////////////////
		writeByteRegister(lcd->file, 0x40, ascii_val);
	}
}
```

Alright! now what we have left is to update our main method:

```
    //Grove LCD has two devices one that drives LCD
	//and a second that drives RGB
	initContext(&rgb, RGB_SLAVE , BUS);
	initContext(&lcd, LCD_SLAVE , BUS);
	////////////////////////////////////////////////////////////
   	
	//init RGB device & set White color
	initLCD(&lcd);
	initRGB(&rgb);	

	writeToLCD(&lcd, ":) HOLO!");
	/*sleep for 5 secs before turning off*/
    sleep(5);
    /*turn off RGB LEDS*/
    turnOffRGB(&rgb);
    printf("\nDONE!\n");
    
```

**clean** , **compile** and run **./lcdtest**

when the programs exits, notice that the LCD still display the blinking cursor as  well the message..  as a challenge write a method that turns off the  LCD.



---


**Answer:**
```
void turnOffLCD(I2CCONTEXT *lcd)
{	
	writeByteRegister(lcd->file, 0x00, LCD_DISPLAYCONTROL );
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );
}
```
by now the whole code should look like this...
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
#include <inttypes.h>

#define RGB_SLAVE       0x62
#define LCD_SLAVE       0x3E
#define BUS             0x01
#define REG_RED         0x04        // pwm2
#define REG_GREEN       0x03        // pwm1
#define REG_BLUE        0x02        // pwm0
#define REG_MODE1       0x00
#define REG_MODE2       0x01
#define REG_OUTPUT      0x08


// commands
#define LCD_CLEARDISPLAY	0x01
#define LCD_RETURNHOME	0x02
#define LCD_ENTRYMODESET	0x04
#define LCD_DISPLAYCONTROL	0x08
#define LCD_CURSORSHIFT	0x10
#define LCD_FUNCTIONSET 0x20
#define LCD_SETCGRAMADDR	0x40
#define LCD_SETDDRAMADDR	0x80

// flags for display entry mode
#define LCD_ENTRYRIGHT	0x00
#define LCD_ENTRYLEFT	0x02
#define LCD_ENTRYSHIFTINCREMENT 0x01
#define LCD_ENTRYSHIFTDECREMENT 0x00

// flags for display on/off control
#define LCD_DISPLAYON	0x04
#define LCD_DISPLAYOFF	0x00
#define LCD_CURSORON	0x02
#define LCD_CURSOROFF	0x00
#define LCD_BLINKON	0x01
#define LCD_BLINKOFF	0x00

// flags for display/cursor shift
#define LCD_DISPLAYMOVE 0x08
#define LCD_CURSORMOVE	0x00
#define LCD_MOVERIGHT	0x04
#define LCD_MOVELEFT	0x00

// flags for function set
#define LCD_8BITMODE	0x10
#define LCD_4BITMODE	0x00
#define LCD_2LINE	0x18
#define LCD_5x8DOTS	0x00
//////////////////////////////

typedef struct
{
    int addr; //i2c address of device
    int file; //reprsents a place to do the i2c read & write
    int adapter_nr; //the bus number of the device
    char filename[20]; //the name of the device
}I2CCONTEXT;

I2CCONTEXT lcd;
I2CCONTEXT rgb;

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
__s32 writeWordRegister(int file, __u8 command, __u16 value)
{
	__s32 res = -1;

	res = i2c_smbus_write_word_data(file, command, value);
	if (res < 0)
	{
	    /* ERROR HANDLING: i2c transaction failed */
		printf("Error writing reg: 0x%x, (errno:%d),%s",
				command,errno,strerror(errno));
	}
	return res;

}
__s32 readWordRegister(int r, int file)
{
	/* Using SMBus commands */
	__u8 reg = r; /* Device register to access */
	__s32 res = -1;
	char buf[10];  
	res = i2c_smbus_read_word_data(file, reg);
	if (res < 0)
	{
	    /* ERROR HANDLING: i2c transaction failed */
		printf("Error reading reg: 0x%x, (errno:%d),%s",
				reg,errno,strerror(errno));
	}
	/*
	else
	{
		printf("result: 0x%x\n",res);
	}
	*/
	return res;
}

__s32 writeByteRegister(int file, __u8 register_, __u8 value)
{
    __s32 res = -1;
    res = i2c_smbus_write_byte_data(file, register_, value);
    if (res < 0)
    {
        /* ERROR HANDLING: i2c transaction failed */
        printf("Error writing byte, (errno:%d),%s\n",errno,
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
void turnOffRGB(I2CCONTEXT *rgb)
{
	setRGBColor(rgb, 0x00, 0x00, 0x00);	
}
void initLCD(I2CCONTEXT *lcd)
{
	writeByteRegister(lcd->file, 0x00, LCD_FUNCTIONSET | LCD_2LINE );
	usleep(100);
	writeByteRegister(lcd->file, 0x00,LCD_DISPLAYCONTROL | LCD_DISPLAYON | LCD_CURSORON | LCD_BLINKOFF );
	usleep(100);
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );
	usleep(4000);
	writeByteRegister(lcd->file, 0x00,LCD_ENTRYMODESET | LCD_ENTRYLEFT | LCD_ENTRYSHIFTDECREMENT );


}
void turnOffLCD(I2CCONTEXT *lcd)
{
	
	writeByteRegister(lcd->file, 0x00, LCD_DISPLAYCONTROL );
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );

}
void clearLCD(I2CCONTEXT *lcd)
{
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );
}
void writeToLCD(I2CCONTEXT *lcd, char *c)
{

	//writeByteRegister(lcd->file, 0x80, 0x48);
	//usleep(4000);
	
	uint ascii_val=0;
	int i;
	int s = strlen(c);
	for(i = 0 ; i < s; i++)
	{
		ascii_val = toascii(c[i]);
		writeByteRegister(lcd->file, 0x40, ascii_val);
	}
}

int main()
{

	//Grove LCD has two devices one that drives LCD
	//and a second that drives RGB
	initContext(&rgb, RGB_SLAVE , BUS);
	initContext(&lcd, LCD_SLAVE , BUS);
	////////////////////////////////////////////////////////////
   	
	//init RGB device & set White color
	initLCD(&lcd);
	initRGB(&rgb);	

	writeToLCD(&lcd, ":) HOLO!");
	/*sleep for 5 secs before turning off*/
    	sleep(5);
    	/*turn off RGB LEDS*/
    	turnOffRGB(&rgb);
    	turnOffLCD(&lcd);
    	printf("\nDONE!\n");


	return 0;
}
```

---
there are things missing like, specify the column and row where we want ot start writing text so it will be left to you to investigicate how to add that functionality.

The final challenge, so you choose to accept it, is to integrate the code that gets the temperature using the i2c barometric sensor.. are you up to the challenge?



---
**Answer (full code)**:
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
#include <inttypes.h>

#define RGB_SLAVE       0x62
#define LCD_SLAVE       0x3E
#define BUS             0x01
#define REG_RED         0x04        // pwm2
#define REG_GREEN       0x03        // pwm1
#define REG_BLUE        0x02        // pwm0
#define REG_MODE1       0x00
#define REG_MODE2       0x01
#define REG_OUTPUT      0x08


// commands
#define LCD_CLEARDISPLAY	0x01
#define LCD_RETURNHOME	0x02
#define LCD_ENTRYMODESET	0x04
#define LCD_DISPLAYCONTROL	0x08
#define LCD_CURSORSHIFT	0x10
#define LCD_FUNCTIONSET 0x20
#define LCD_SETCGRAMADDR	0x40
#define LCD_SETDDRAMADDR	0x80

// flags for display entry mode
#define LCD_ENTRYRIGHT	0x00
#define LCD_ENTRYLEFT	0x02
#define LCD_ENTRYSHIFTINCREMENT 0x01
#define LCD_ENTRYSHIFTDECREMENT 0x00

// flags for display on/off control
#define LCD_DISPLAYON	0x04
#define LCD_DISPLAYOFF	0x00
#define LCD_CURSORON	0x02
#define LCD_CURSOROFF	0x00
#define LCD_BLINKON	0x01
#define LCD_BLINKOFF	0x00

// flags for display/cursor shift
#define LCD_DISPLAYMOVE 0x08
#define LCD_CURSORMOVE	0x00
#define LCD_MOVERIGHT	0x04
#define LCD_MOVELEFT	0x00

// flags for function set
#define LCD_8BITMODE	0x10
#define LCD_4BITMODE	0x00
#define LCD_2LINE	0x18
#define LCD_5x8DOTS	0x00
//////////////////////////////
//

//Barometer Registers
#define BAROMETER_SLAVE		0x60
#define MPL3115A2_WHO_AM_I      0x0c	//Device Identification Register
#define MPL3115A2_SYSMOD	0x11	//System Mode Register || Current system mode
#define MPL3115A2_CTRL_REG1	0x26	//Control Register 1 || Modes, Oversampling
#define MPL3115A2_CTRL_REG2	0x27	//Control Register 2 ||	Acquisition time step	 
#define MPL3115A2_CTRL_REG3	0x28	//Control Register 3 || Interrupt pin configuration
#define MPL3115A2_CTRL_REG4	0x29	//Control Register 4 || Interrupt enables
#define MPL3115A2_CTRL_REG5	0x2A	//Control Register 5 || Interrupt output pin assignment
#define MPL3115A2_STATUS	0x00	//Sensor Status Register 
#define MPL3115A2_OFF_T		0x2C	//Temperature Data Offset Register
#define MPL3115A2_BUS		0x01	//I2C bus where the device is connected
#define MPL3115A2_SLAVE		0x60	//I2C slave address of the device
#define MPL3115A2_OUT_T_MSB	0x04
#define MPL3115A2_OUT_T_LSB	0x05
#define MPL3115A2_DR_STATUS	0x06	//
#define MPL3115A2_OST_MASK	0x02



//////////////////////////////////////////////////////////

typedef struct
{
    int addr; //i2c address of device
    int file; //reprsents a place to do the i2c read & write
    int adapter_nr; //the bus number of the device
    char filename[20]; //the name of the device
}I2CCONTEXT;

I2CCONTEXT lcd;
I2CCONTEXT rgb;
I2CCONTEXT barometer;

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
__s32 writeWordRegister(int file, __u8 command, __u16 value)
{
	__s32 res = -1;

	res = i2c_smbus_write_word_data(file, command, value);
	if (res < 0)
	{
	    /* ERROR HANDLING: i2c transaction failed */
		printf("Error writing reg: 0x%x, (errno:%d),%s",
				command,errno,strerror(errno));
	}
	return res;

}
__s32 readWordRegister(int r, int file)
{
	/* Using SMBus commands */
	__u8 reg = r; /* Device register to access */
	__s32 res = -1;
	char buf[10];  
	res = i2c_smbus_read_word_data(file, reg);
	if (res < 0)
	{
	    /* ERROR HANDLING: i2c transaction failed */
		printf("Error reading reg: 0x%x, (errno:%d),%s",
				reg,errno,strerror(errno));
	}
	/*
	else
	{
		printf("result: 0x%x\n",res);
	}
	*/
	return res;
}

__s32 writeByteRegister(int file, __u8 register_, __u8 value)
{
    __s32 res = -1;
    res = i2c_smbus_write_byte_data(file, register_, value);
    if (res < 0)
    {
        /* ERROR HANDLING: i2c transaction failed */
        printf("Error writing byte, (errno:%d),%s\n",errno,
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
void turnOffRGB(I2CCONTEXT *rgb)
{
	setRGBColor(rgb, 0x00, 0x00, 0x00);	
}
void initLCD(I2CCONTEXT *lcd)
{
	writeByteRegister(lcd->file, 0x00, LCD_FUNCTIONSET | LCD_2LINE );
	usleep(100);
	writeByteRegister(lcd->file, 0x00,LCD_DISPLAYCONTROL | LCD_DISPLAYON | LCD_CURSORON | LCD_BLINKOFF );
	usleep(100);
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );
	usleep(4000);
	writeByteRegister(lcd->file, 0x00,LCD_ENTRYMODESET | LCD_ENTRYLEFT | LCD_ENTRYSHIFTDECREMENT );


}
void turnOffLCD(I2CCONTEXT *lcd)
{
	
	writeByteRegister(lcd->file, 0x00, LCD_DISPLAYCONTROL );
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );

}
void clearLCD(I2CCONTEXT *lcd)
{
	writeByteRegister(lcd->file, 0x00, LCD_CLEARDISPLAY );
}
void writeToLCD(I2CCONTEXT *lcd, char *c)
{

	//writeByteRegister(lcd->file, 0x80, 0x48);
	//usleep(4000);
	
	uint ascii_val=0;
	int i;
	int s = strlen(c);
	for(i = 0 ; i < s; i++)
	{
		ascii_val = toascii(c[i]);
		writeByteRegister(lcd->file, 0x40, ascii_val);
	}
}

void doOneShot(I2CCONTEXT *barometer)
{

	printf("Doing oneShot\n");
	__s32 ctrl_reg1 = readWordRegister(MPL3115A2_CTRL_REG1,barometer->file);
	ctrl_reg1 |= (1<<1);
	writeWordRegister(barometer->file, MPL3115A2_CTRL_REG1, ctrl_reg1);
} 
float getTemp(I2CCONTEXT *barometer)
{


	float temperature = 0.0;
	__s32 msb = 0;
	__s32 lsb = 0;
	__s32 ctrl_reg1 = readWordRegister(MPL3115A2_CTRL_REG1,barometer->file);
	printf("OST STATUS: 0x%x\n",ctrl_reg1 & 0x02);
	if((ctrl_reg1 & 0x02) == 0)
	{
		printf("%s\n", "no oneShoot in progress..");
		doOneShot(barometer);
		int contador = 0;
		while((ctrl_reg1 & 0x02) == 0)
		{
			ctrl_reg1 = readWordRegister(MPL3115A2_CTRL_REG1,barometer->file);
		}
		
		int msb = (readWordRegister(MPL3115A2_OUT_T_MSB,barometer->file)) & 255 ;
		int lsb = readWordRegister(MPL3115A2_OUT_T_LSB,barometer->file);
		float templsb = (lsb>>4)/16.0; //temp, fraction of a degree
				
		temperature = (float)msb+templsb;
		printf("Temperature: %f\n",temperature);
		return temperature;
		

	}
	else
	{
		printf("%s\n", "in progress...");
		return 0;
	}
	
}
int main()
{

    /*
	we pass a reference to the rgb context variable
	the i2c address of the rgb controller
	and the BUS which should be 1
    */
    initContext(&rgb, RGB_SLAVE , BUS);
    initContext(&lcd, LCD_SLAVE , BUS);
    initContext(&barometer, BAROMETER_SLAVE , BUS);
    /*inits the RGB controller and sets its color to White*/
    initRGB(&rgb);
    initLCD(&lcd);

	int contador = 20;
	char t[80];
	while(contador > 0)
	{
		clearLCD(&lcd);
		sprintf(t, "Temp:%.2f *C", getTemp(&barometer));
		writeToLCD(&lcd, t);
		contador--;
		/*sleep for 1sec*/
		sleep(1);
		
	}
	
	    
    

    /*turn off RGB LEDS*/
    turnOffRGB(&rgb);
    turnOffLCD(&lcd);
    printf("\nDONE!\n");
    	
    return 0;
}

```

---
