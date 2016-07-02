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
#define BUS             0x06
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

And to read a register we can use:

```c
i2c_smbus_read_word_data(int file, __u8 register, __u8 value);
```

Lets create two methods so we can handle any errors and return the error code in case something goes wrong.

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

So far your **lcd.c** sourcecode should look like this:

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
#define BUS             0x06
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
root@edison:~/.../I2CDL# ./lcdtest
                                                                                           
DONE! 
```

There are things missing like, specify the column and row where we want to start writing text so it will be left to you to investigate how to add that functionality.

