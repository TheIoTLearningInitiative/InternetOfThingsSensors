# Communication Protocols

> A protocol defines how a sensor talks to the microcontroller board, such as Arduino, Raspberry Pi, Edison or Galileo (just to name few). The protocol defines how the wires should be connected and how your code should ask for measurements. Even though there is a staggering amount of different sensors, there is a limited number of popular protocols. [References](References.md)

# General Concepts

> Instruction Register (IR) - the IR stores instruction codes

> Data Register (DR) - the DR temporarily stores data to be written into DDRAM or CGRAM and temporarily stores data to be read from DDRAM or CGRAM. The DR is also used for data storage when reading data from DDRAM or CGRAM.

> Register Selector (RS) selects the Instruction Register (IR) or Data Register (DR)

# Serial Communication / Universal Asynchronous Receiver/Transmitter

> A universal asynchronous receiver/transmitter, abbreviated UART, is a computer hardware device that translates data between characters (usually bytes) in a computer and an asynchronous serial communication format that encapsulates those characters between start bits and stop bits. UARTs are commonly used in conjunction with communication standards such as TIA (formerly EIA) RS-232, RS-422 or RS-485. [Wikipedia](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver/transmitter)

> Embedded electronics is all about interlinking circuits (processors or other integrated circuits) to create a symbiotic system. In order for those individual circuits to swap their information, they must share a common communication protocol. Hundreds of communication protocols have been defined to achieve this data exchange, and, in general, each can be separated into one of two categories: parallel or serial. [Sparkfun Learn about Serial Communication](https://learn.sparkfun.com/tutorials/serial-communication) 

# IR

> IR, or infrared, communication is a common, inexpensive, and easy to use wireless communication technology. IR light is very similar to visible light, except that it has a slightlty longer wavelength. This means IR is undetectable to the human eye - perfect for wireless communication. For example, when you hit a button on your TV remote, an IR LED repeatedly turns on and off, 38,000 time a second, to transmit information (like volume or channel control) to an IR photo sensor on your TV.[Learn about IR](https://learn.sparkfun.com/tutorials/ir-communication)

# PWM

> Pulse width modulation (PWM) is a fancy term for describing a type of digital signal. Pulse width modulation is used in a variety of applications including sophisticated control circuitry. A common way we use them here at SparkFun is to control dimming of RGB LEDs or to control the direction of a servo motor [Learn about PWM](https://learn.sparkfun.com/tutorials/pulse-width-modulation?_ga=1.187957866.310968540.1461470817)

