# Prerequisites

# Pre-Work

```sh
root@edison:~# git clone https://github.com/YourUserName/TheIoTLearningInitiative.git
Cloning into 'TheIoTLearningInitiative'...
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (4/4), done.
Checking connectivity... done.
root@edison:~#
```

```sh
root@edison:~# cd TheIoTLearningInitiative/
root@edison:~/TheIoTLearningInitiative# mkdir Sensors
root@edison:~/TheIoTLearningInitiative# cd
root@edison:~#
```

# Equipment

* Laptop \(Linux or Windows\)
* Internet Connection
* Micro USB Cable
* Arduino 101
* Linux Based Board Development \(Intel Edison, Intel Galileo\)
* [Grove Indoor Environment Kit for Intel® Edison](https://www.seeedstudio.com/item_detail.html?p_id=2427)

  * [Grove - LCD RGB Backlight \(I2C Communication Protocol Device\)](http://www.seeedstudio.com/wiki/Grove_-_LCD_RGB_Backlight)
    ![](http://www.seeedstudio.com/wiki/images/thumb/0/03/Serial_LEC_RGB_Backlight_Lcd.jpg/500px-Serial_LEC_RGB_Backlight_Lcd.jpg)
  * [Grove - Barometer Sensor \(I2C Communication Protocol Device\)](http://www.seeedstudio.com/wiki/Grove_-_Barometer_Sensor)

    ![](http://www.seeedstudio.com/wiki/images/thumb/e/e7/Grove-Barometer.jpg/300px-Grove-Barometer.jpg)

# Intel Edison Boot Up

```sh
Poky (Yocto Project Reference Distro) 1.7.2 edison ttyMFD2

    edison login: root
    root@edison:~# configure_edison --wifi
    ...
    root@edison:~#
```

# Kernel Development Environment: Module Native Compilation

## Kernel Headers Setup

Taken from [SourceCodeCat Github](https://github.com/SourceCodeCat/IoTDownloads)

```sh
root@edison:~# cd
root@edison:~# wget https://github.com/SourceCodeCat/IoTDownloads/raw/master/linux-headers-3.10.17-poky-edison_3.10.17-poky-edison-1_i386.deb
```

Under Intel Edison

```sh
root@edison:~# ar x linux-headers-3.10.17-poky-edison_3.10.17-poky-edison-1_i386.deb
```

```sh
root@edison:~# ls data.tar.gz 
data.tar.gz
```

```sh
root@edison:~# tar -xvf data.tar.gz
...
./usr/src/linux-headers-3.10.17-poky-edison/drivers/iio/frequency/Makefile
./usr/src/linux-headers-3.10.17-poky-edison/drivers/iio/frequency/Kconfig
./lib/
./lib/modules/
./lib/modules/3.10.17-poky-edison/
./lib/modules/3.10.17-poky-edison/build
```

```sh
root@edison:~# uname -a
Linux edison 3.10.98-poky-edison+ #1 SMP PREEMPT Mon Jun 6 14:32:08 PDT 2016 i6x
root@edison:~#
```

## Version 3.10.98-poky-edison+

```sh
root@edison:~# cd ~/usr/src/
root@edison:~# ln -s linux-headers-3.10.17-poky-edison linux-headers-3.10.98-poky-edison
root@edison:~/usr/src# ls ~/usr/src/                                            
linux-headers-3.10.17-poky-edison  linux-headers-3.10.98-poky-edison
```

```sh
root@edison:~# nano ~/usr/src/linux-headers-3.10.98-poky-edison/include/generated/utsrelease.h
    #define UTS_RELEASE "3.10.98-poky-edison+"
    <Save Changes>
```

```sh
root@edison:~# cd /lib/modules/3.10.98-poky-edison+
```

```sh
root@edison:/lib/modules/3.10.98-poky-edison+# ls
    extra                modules.builtin.bin  modules.softdep
    kernel               modules.dep          modules.symbols
    modules.alias        modules.dep.bin      modules.symbols.bin
    modules.alias.bin    modules.devname
    modules.builtin      modules.order
```

```sh
root@edison:/lib/modules/3.10.98-poky-edison+# ln -s /home/root/usr/src/linux-headers-3.10.98-poky-edison build
```

```sh
root@edison:/lib/modules/3.10.98-poky-edison+# cd
    root@edison:~#
```



