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
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# ls                                     
JHD1313M2.c  JHD1313M2.h  Makefile                                                         
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# 
```

```sh
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# make
make -C /lib/modules/3.10.98-poky-edison+/build M=/home/root/TheIoTLearningInitiative/Senss
make[1]: Entering directory '/home/root/usr/src/linux-headers-3.10.17-poky-edison'
  CC [M]  /home/root/TheIoTLearningInitiative/Sensors/LKM/JHD1313M2.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/root/TheIoTLearningInitiative/Sensors/LKM/JHD1313M2.mod.o
  LD [M]  /home/root/TheIoTLearningInitiative/Sensors/LKM/JHD1313M2.ko
make[1]: Leaving directory '/home/root/usr/src/linux-headers-3.10.17-poky-edison'
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# 
```