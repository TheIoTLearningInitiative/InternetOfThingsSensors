# Display Color

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

Now lets call init RGB from our main() by passing the reference to our RGB context wait about 5 seconds and then exit turning off the color leds:

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
