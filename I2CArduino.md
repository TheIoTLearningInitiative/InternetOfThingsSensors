# Arduino


# Arduino 101

## Master Mode


#include <Wire.h>

```c
void setup() {
  Wire.begin();        // join i2c bus (address optional for master)
  Serial.begin(9600);  // start serial for output
}

void loop() {
  Wire.requestFrom(8, 6);    // request 6 bytes from slave device #8

  while (Wire.available()) { // slave may send less than requested
    char c = Wire.read(); // receive a byte as character
    Serial.print(c);         // print the character
  }

  delay(500);
}
```

## Slave Mode

- [Arduino 101: No I2C Slave mode support in wire.h](https://github.com/01org/corelibs-arduino101/issues/112)


