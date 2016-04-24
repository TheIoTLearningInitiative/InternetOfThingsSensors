# Sensor Protocols



A protocol defines how a sensor talks to the microcontroller board, such as Arduino, Raspberry Pi, Edison or Galileo (just to name few). The protocol defines how the wires should be connected and how your code should ask for measurements.
Even though there is a staggering amount of different sensors, there is a limited number of popular protocols:


#### SPI
Serial Peripheral Interface (SPI) is an interface bus commonly used to send data between microcontrollers and small peripherals such as shift registers, sensors, and SD cards. It uses separate clock and data lines, along with a select line to choose the device you wish to talk to.  [Learn about SPI](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi)

####I2C
The Inter-integrated Circuit (I2C) Protocol is a protocol intended to allow multiple “slave” digital integrated circuits (“chips”) to communicate with one or more “master” chips. Like the Serial Peripheral Interface (SPI), it is only intended for short distance communications within a single device. Like Asynchronous Serial Interfaces (such as RS-232 or UARTs), it only requires two signal wires to exchange information. [Learn about I2C](https://learn.sparkfun.com/tutorials/i2c)

####Serial Communication

Embedded electronics is all about interlinking circuits (processors or other integrated circuits) to create a symbiotic system. In order for those individual circuits to swap their information, they must share a common communication protocol. Hundreds of communication protocols have been defined to achieve this data exchange, and, in general, each can be separated into one of two categories: parallel or serial. [Learn about Serial Communication](https://learn.sparkfun.com/tutorials/serial-communication) 
