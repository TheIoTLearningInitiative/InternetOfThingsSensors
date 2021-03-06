# Contributions

You can contribute with source code to UPM community! 

# UPM

## Porting

> Porting arduino libraries to libmraa as UPM libraries is usually fairly easy. The issues typically come from misunderstanding of how a non real time OS deals with interrupts and timers. It also highly depends on the sensor. [Porting a module from Arduino](https://github.com/intel-iot-devkit/upm/blob/master/docs/porting.md)

> A concrete example is explained in detail on @ref [max31855](https://github.com/intel-iot-devkit/upm/blob/master/docs/max31855.md)

## Naming Conventions

> UPM attempts to follow a clear naming pattern. Modules should be sensibly named and then placed in ${libdir}/upm and headers in ${includedir}/upm, all modules should be prefixed with libupm-. The upm_module_init will automatically name python UPM modules as pyupm_ and javascript modules as jsupm_. For example for src/grove/ the library built will be libupm-grove.so, the python module pyupm_grove and the js module jsupm_grove. [Naming a module](https://github.com/intel-iot-devkit/upm/blob/master/docs/naming.md)

## Contributions

> Here are the rules of contribution... [Contributing a module](https://github.com/intel-iot-devkit/upm/blob/master/docs/contributions.md)

## Documentation

> It is highly encouraged to provide at least some basic documentation for the sensors that you want to add to UPM... [Writing sensor documentation](https://github.com/intel-iot-devkit/upm/blob/master/docs/documentation.md)