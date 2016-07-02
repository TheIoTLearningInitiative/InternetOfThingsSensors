# Linux Kernel Module

# I'm a Pro!

I'm a pro! (turning our sensor code into a module). The full code for our LCD driver is here:

From [SourceCodeCat Github](https://github.com/SourceCodeCat/IoTCodeExamples/)
Implementation File
https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/JHD1313M2.c
Header
https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/JHD1313M2.h

Make file:
https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/Makefile

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
root@edison:~/.../LKM# wget https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/JHD1313M2.c
```

```sh
root@edison:~/.../LKM# wget https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/JHD1313M2.h
```

```sh
root@edison:~/.../LKM# wget https://github.com/SourceCodeCat/IoTCodeExamples/blob/master/I2C/Makefile
```

```sh
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# ls                                     
JHD1313M2.c  JHD1313M2.h  Makefile                                                         
root@edison:~/TheIoTLearningInitiative/Sensors/LKM# 
```