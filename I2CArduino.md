# Arduino


# Arduino 101

## Master Mode


```c
#include <Wire.h>

void setup() {
  Wire.begin();
  Serial.begin(9600);
}

void loop() {
  Wire.requestFrom(20, 6);

  while (Wire.available()) {
    char c = Wire.read();
    Serial.print(c);
  }

  delay(500);
}
```

## Slave Mode

- [Arduino 101: No I2C Slave mode support in wire.h](https://github.com/01org/corelibs-arduino101/issues/112)


