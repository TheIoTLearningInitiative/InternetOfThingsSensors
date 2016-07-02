# Sensor Protocols

> A protocol defines how a sensor talks to the microcontroller board, such as Arduino, Raspberry Pi, Edison or Galileo (just to name few). The protocol defines how the wires should be connected and how your code should ask for measurements. Even though there is a staggering amount of different sensors, there is a limited number of popular protocols. [References](References.md)

Here we show the most common ones:

# Serial Peripheral Communication

> Serial Peripheral Interface (SPI) is an interface bus commonly used to send data between microcontrollers and small peripherals such as shift registers, sensors, and SD cards. It uses separate clock and data lines, along with a select line to choose the device you wish to talk to.  [Sparkfun Learn about Serial Peripheral Interface (SPI)](https://learn.sparkfun.com/tutorials/serial-peripheral-interface-spi)

# I2C

> The Inter-integrated Circuit (I2C) Protocol is a protocol intended to allow multiple “slave” digital integrated circuits (“chips”) to communicate with one or more “master” chips. Like the Serial Peripheral Interface (SPI), it is only intended for short distance communications within a single device. Like Asynchronous Serial Interfaces (such as RS-232 or UARTs), it only requires two signal wires to exchange information. [Learn about I2C](https://learn.sparkfun.com/tutorials/i2c)

- [I2C Hacking Demystified](http://events.linuxfoundation.org/sites/events/files/slides/ELC%202016%20-%20I2C%20hacking%20demystified_0.pdf)

# Serial Communication
> Embedded electronics is all about interlinking circuits (processors or other integrated circuits) to create a symbiotic system. In order for those individual circuits to swap their information, they must share a common communication protocol. Hundreds of communication protocols have been defined to achieve this data exchange, and, in general, each can be separated into one of two categories: parallel or serial. [Learn about Serial Communication](https://learn.sparkfun.com/tutorials/serial-communication) 

# Universal Asynchronous Receiver/Transmitter
> A universal asynchronous receiver/transmitter, abbreviated UART, is a computer hardware device that translates data between characters (usually bytes) in a computer and an asynchronous serial communication format that encapsulates those characters between start bits and stop bits. UARTs are commonly used in conjunction with communication standards such as TIA (formerly EIA) RS-232, RS-422 or RS-485. [Wikipedia](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver/transmitter)

# IR

> IR, or infrared, communication is a common, inexpensive, and easy to use wireless communication technology. IR light is very similar to visible light, except that it has a slightlty longer wavelength. This means IR is undetectable to the human eye - perfect for wireless communication. For example, when you hit a button on your TV remote, an IR LED repeatedly turns on and off, 38,000 time a second, to transmit information (like volume or channel control) to an IR photo sensor on your TV.[Learn about IR](https://learn.sparkfun.com/tutorials/ir-communication)

# PWM

> Pulse width modulation (PWM) is a fancy term for describing a type of digital signal. Pulse width modulation is used in a variety of applications including sophisticated control circuitry. A common way we use them here at SparkFun is to control dimming of RGB LEDs or to control the direction of a servo motor [Learn about PWM](https://learn.sparkfun.com/tutorials/pulse-width-modulation?_ga=1.187957866.310968540.1461470817)

