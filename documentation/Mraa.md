# MRAA

> Low Level Skeleton Library for IO Communication on GNU/Linux platforms

> C/C++ library with bindings to JavaScript and Python to interface with the I/O on the Intel® Galileo board, Intel® Edison board, and other platforms with a structured and sane API where port names/numbering matches the board that you are on. Use of MRAA does not tie you to specific hardware, with board detection done at runtime, you can create portable code that works across multiple platforms.

- [MRAA Github](https://github.com/intel-iot-devkit/mraa)

- AIO Sensors requiring an ADC value to be read
- I2C Modules using the i2c bus
- SPI Modules using the SPI bus
- GPIO Modules using GPIOs directly
- PWM Modules using a PWM capable GPIO pin
- UART Modules using a serial connection (RX/TX)

# Manual Installation, Latest Github Code Version

## Yocto

```sh
    root@edison:~# git clone https://github.com/intel-iot-devkit/mraa.git
    root@edison:~# mkdir mraa/build && cd $_
    root@edison:~# cmake ..
    root@edison:~# make
    root@edison:~# make install
    root@edison:~# nano /etc/ld.so.conf
    /usr/local/lib/i386-linux-gnu/
    root@edison:~# ldconfig
    root@edison:~# ldconfig -p | grep mraa
    root@edison:~# export PYTHONPATH=$PYTHONPATH:$(dirname $(find /usr/local -name mraa.py))
    root@edison:~# nano ~/.bashrc
    export PYTHONPATH=$PYTHONPATH:$(dirname $(find /usr/local -name mraa.py))
```

## Ubilinux

```sh
    root@ubilinux:~# apt-get update
    root@ubilinux:~# apt-cache search pcre
    root@ubilinux:~# apt-get install libpcre3-dev
    root@ubilinux:~# apt-get install git
    root@ubilinux:~# apt-get install cmake
    root@ubilinux:~# apt-get install python-dev
    root@ubilinux:~# apt-get install swig
    root@ubilinux:~# git clone https://github.com/intel-iot-devkit/mraa.git
    root@ubilinux:~# mkdir mraa/build && cd $_
    root@ubilinux:~# cmake .. -DBUILDSWIGNODE=OFF
    root@ubilinux:~# make
    root@ubilinux:~# make install
    root@ubilinux:~# cd
    root@ubilinux:~# nano /etc/ld.so.conf
    /usr/local/lib/i386-linux-gnu/
    root@ubilinux:~# ldconfig
    root@ubilinux:~# ldconfig -p | grep mraa
    root@ubilinux:~# export PYTHONPATH=$PYTHONPATH:$(dirname $(find /usr/local -name mraa.py))
    root@ubilinux:~# nano ~/.bashrc
    export PYTHONPATH=$PYTHONPATH:$(dirname $(find /usr/local -name mraa.py))
```

Based on [Installing libmraa on Ubilinux for Edison](https://learn.sparkfun.com/tutorials/installing-libmraa-on-ubilinux-for-edison)

# Upgrade via Opkg

```sh
root@edison:~# opkg list-installed | grep mraa
mraa - 1.0.0-r0
mraa-dev - 1.0.0-r0
mraa-doc - 1.0.0-r0
```

```sh
root@edison:~# opkg install mraa                                                
Package mraa (1.0.0-r0) installed in root is up to date.                        
root@edison:~# 
```

# MRAA Labs

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

from there you can see that we need to installed at least version 0.7.0 of mraa. In the same fashion you can check the version of the installed libmraa0 library.

This basic mraa example, can be used to verify it is correctly installed and found by the compiler

```sh
root@edison:~# nano mraa.c
```

```c
#include "mraa.h"

int main(int argc, char** argv)
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
