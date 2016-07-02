# MRAA

##libmraa - Low Level Skeleton Library for Communication on GNU/Linux platforms

Libmraa is a C/C++ library with bindings to javascript & python to interface with the IO on Galileo, Edison & other platforms, with a structured and sane API where port names/numbering matches the board that you are on. Use of libmraa does not tie you to specific hardware with board detection done at runtime you can create portable code that will work across the supported platforms.

The intent is to make it easier for developers and sensor manufacturers to map their sensors & actuators on top of supported hardware and to allow control of low level communication protocol by high level languages & constructs.

Note: using mraa and UPM requires to be aware of the versions installed, since there is a version dependency of UPM over MRAA. 

To find out know the mraa version that needs to be installed in order to use UPM, type the following in your edison terminal:

```sh
root@edison:~# opkg info upm
```

Something like this is printed in the screen:

```sh
Package: upm                                                                    
Version: 0.5.1-r0                                                               
Depends: libgcc1 (>= 4.9.1), libpython2.7-1.0 (>= 2.7.3), libc6 (>= 2.20), pyth)
Status: unknown ok not-installed                                                
Section: libs                                                                   
Architecture: core2-32                                                          
Maintainer: Poky <poky@yoctoproject.org>                                        
MD5Sum: eb8bbbf6d30c35a79864955914029134                                        
Size: 25851484                                                                  
Filename: upm_0.5.1-r0_core2-32.ipk                                             
Source: git://github.com/intel-iot-devkit/upm.git;protocol=git;tag=v0.5.1       
Description: Sensor/Actuator repository for Mraa  Sensor/Actuator repository fo.
                                                                                
Package: upm                                                                    
Version: 0.7.0-r0                                                               
Depends: libgcc1 (>= 4.9.1), libpython2.7-1.0 (>= 2.7.3), libc6 (>= 2.20), pyth)
Status: install ok installed                                                    
Section: libs                                                                   
Architecture: core2-32                                                          
Maintainer: Poky <poky@yoctoproject.org>                                        
MD5Sum: d50038c9eb7d197267a2aa9c4531ad20                                        
Size: 33113334                                                                  
Filename: upm_0.7.0-r0_core2-32.ipk                                             
Source: git://github.com/intel-iot-devkit/upm.git;protocol=git;tag=v0.7.0       
Description: Sensor/Actuator repository for Mraa  Sensor/Actuator repository fo.
Installed-Time: 1465260692                                                      
                   
root@edison:~# 
```

from there you can see that we need to installed at least version 0.6.2 of mraa. In the same fashion you can check the version of the installed libmraa0 library.

simple mraa example, can be used to verify it is correctly installed and found by the compiler

```
#include "mraa.h"
int
main(int argc, char** argv)
{
    char* board_name = mraa_get_platform_name();
    int i2c_bus, i, i2c_adapter;
    fprintf(stdout, "hello mraa\n Version: %s\n Running on %s\n", mraa_get_version(), board_name);
    mraa_deinit();
    return MRAA_SUCCESS;
}
```


How to compile?..let's say you have the code above in a file named HelloMRAA.cpp, inside your edison, now from the Edison terminal type:

    g++ -lmraa HelloMRAA.cpp -o HelloMRAA
    
now if you run the program typing:

    ./HelloMRAA

you will see something like this:

```
hello mraa
 Version: v0.7.3
 Running on Intel Edison

```
