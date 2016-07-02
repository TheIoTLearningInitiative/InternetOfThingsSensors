# Linux Kernel Module

# I'm a Pro!

I'm a pro! (turning our sensor code into a module). The full code for our LCD driver is here:

From [SourceCodeCat Github](https://github.com/SourceCodeCat/IoTCodeExamples/)

- Implementation File: https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/JHD1313M2.c
- Header: https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/JHD1313M2.h
- Makefile: https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/Makefile

```sh
root@edison:~# cd
root@edison:~# cd TheIoTLearningInitiative/Sensors
root@edison:~/TheIoTLearningInitiative/Sensors# 
```

```sh
root@edison:~/TheIoTLearningInitiative/Sensors# mkdir LKM
root@edison:~/TheIoTLearningInitiative/Sensors# cd LKM/
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# 
```

```sh
root@edison:~/.../LKM# wget https://raw.githubusercontent.com/SourceCodeCat/IoTCodeExamples/master/I2C/JHD1313M2.c
```

```sh
root@edison:~/.../LKM# wget https://raw.githubusercontent.com/SourceCodeCat/IoTCodeExamples/master/I2C/JHD1313M2.h
```

```sh
root@edison:~/.../LKM# nano Makefile
```

```sh
obj-m += JHD1313M2.o

all:
        make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
clean:
        make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

```sh
root@edison:~/.../LKM# ls                                     
JHD1313M2.c  JHD1313M2.h  Makefile                                                         
root@edison:~/.../LKM#  
```

```sh
root@edison:~/.../LKM# make
make -C /lib/modules/3.10.98-poky-edison+/build M=/home/root/TheIoTLearningInitiative/Senss
make[1]: Entering directory '/home/root/usr/src/linux-headers-3.10.17-poky-edison'
  CC [M]  /home/root/TheIoTLearningInitiative/Sensors/LKM/JHD1313M2.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/root/TheIoTLearningInitiative/Sensors/LKM/JHD1313M2.mod.o
  LD [M]  /home/root/TheIoTLearningInitiative/Sensors/LKM/JHD1313M2.ko
make[1]: Leaving directory '/home/root/usr/src/linux-headers-3.10.17-poky-edison'
root@edison:~/.../LKM#  
```

```sh
root@edison:~/.../LKM# ls  
JHD1313M2.c  JHD1313M2.ko     JHD1313M2.mod.o  Makefile        modules.order
JHD1313M2.h  JHD1313M2.mod.c  JHD1313M2.o      Module.symvers
root@edison:~/.../LKM# 
```

```sh
root@edison:~/.../LKM# sudo insmod JHD1313M2.ko
```

```sh
root@edison:~/.../LKM# dmesg
[ 1363.707731] JHD1313M2_init: Obtaining adapter from bus 6                                
[ 1363.707760] JHD1313M2_init: creating new device...                                      
[ 1363.708163] JHD1313M2_init: trying to add the device driver...                          
[ 1363.708237] JHD1313M2_RGB_probe: trying to probe the device (JHD1313M2_RGB)...          
[ 1363.719955] JHD1313M2_LCD_probe: trying to probe the device (JHD1313M2_LCD)...          
[ 1363.808522] JHD1313M2_init: creating sysfs access...
```



```sh
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# ls /sys/kernel/JHD1313M2/
lcd_text  rgb_b  rgb_g  rgb_r
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# 
```

```sh
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# echo "from here now" > /sys/kernel/JHD1313M2/lcd_text
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# 
```
