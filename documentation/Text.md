# Text


Now lets write some methods to drive the LCD controller, first of all the JHD1313 datasheet (Grove for some reason puts a link to the datasheet of JHD1214) is not too clear, but after some time you will realize that to init the device you can write to register 0x00 if you want to write to CGRAM AD you need to write to register 0x40 and to write information to DDRAM you need to write to register 0x80 (take a look to the [JHD1214 data sheet](http://www.seeedstudio.com/wiki/images/0/03/JHD1214Y_YG_1.0.pdf)).

So lets start adding some defines we will need:

```c
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

```c
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

```c
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

```c
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

```sh
root@edison:~/.../I2CDL# make clean
rm lcd
```

```sh
root@edison:~/.../I2CDL# make
gcc -O lcd.c -o lcdtest
```

```sh
root@edison:~/.../I2CDL# ./lcdtest
                                                                                           
DONE! 
```

