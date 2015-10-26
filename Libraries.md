# Libraries: using mraa / UPM


##UPM

Where are the UPM includes in Edison?

here: <font color="green">/usr/include/upm</font>, if you want to see the full list of headers supported by your installed upm, type this:  <font color="green">ls -l /usr/include/upm</font>, which list all the headers, in this case the following header are the ones supported in **Version: 0.3.2** :

```
-rw-r--r--    1 root     root          3192 Aug 13 00:23 a110x.h
-rw-r--r--    1 root     root          2933 May 19 16:28 aci_queue.h
-rw-r--r--    1 root     root          2157 May 19 16:28 aci_setup.h
-rw-r--r--    1 root     root          2723 May 19 16:28 acilib.h
-rw-r--r--    1 root     root          3154 Aug 13 00:23 ad8232.h
-rw-r--r--    1 root     root          6787 Aug 13 00:23 adafruitms1438.h
-rw-r--r--    1 root     root          3490 Aug 13 00:23 adafruitss.h
-rw-r--r--    1 root     root          9885 Aug 13 00:23 adc121c021.h
-rw-r--r--    1 root     root          6140 Aug 13 00:23 adis16448.h
-rw-r--r--    1 root     root          4024 Aug 13 00:23 adxl335.h
-rw-r--r--    1 root     root          3148 Aug 13 00:23 adxl345.h
-rw-r--r--    1 root     root          6824 Aug 13 00:23 ak8975.h
-rw-r--r--    1 root     root          5773 Aug 13 00:23 am2315.h
-rw-r--r--    1 root     root          2213 Aug 13 00:23 apds9002.h
-rw-r--r--    1 root     root          7703 Aug 13 00:23 at42qt1070.h
-rw-r--r--    1 root     root          2202 Aug 13 00:23 biss0001.h
-rw-r--r--    1 root     root          5604 Aug 13 00:23 bmpx8x.h
-rw-r--r--    1 root     root          3781 Aug 13 00:23 buzzer.h
-rw-r--r--    1 root     root          3342 Aug 13 00:23 cjq4435.h
-rw-r--r--    1 root     root          5360 Aug 13 00:23 ds1307.h
-rw-r--r--    1 root     root          5224 Aug 13 00:23 eboled.h
-rw-r--r--    1 root     root          4041 Aug 13 00:23 ecs1030.h
-rw-r--r--    1 root     root          3276 Aug 13 00:23 enc03r.h
-rw-r--r--    1 root     root          2107 Aug 13 00:23 es08a.h
-rw-r--r--    1 root     root          2336 Aug 13 00:23 flex.h
-rw-r--r--    1 root     root          3420 Aug 13 00:23 gas.h
-rw-r--r--    1 root     root          6921 Aug 13 00:23 gfx.h
-rw-r--r--    1 root     root          2533 Aug 13 00:23 gp2y0a.h
-rw-r--r--    1 root     root         10870 Aug 13 00:23 grove.h
-rw-r--r--    1 root     root          3439 Aug 13 00:23 grovecircularled.h
-rw-r--r--    1 root     root          2240 Aug 13 00:23 grovecollision.h
-rw-r--r--    1 root     root          3061 Aug 13 00:23 groveehr.h
-rw-r--r--    1 root     root          2199 Aug 13 00:23 groveeldriver.h
-rw-r--r--    1 root     root          2215 Aug 13 00:23 groveelectromagnet.h
-rw-r--r--    1 root     root          2242 Aug 13 00:23 groveemg.h
-rw-r--r--    1 root     root          2367 Aug 13 00:23 grovegsr.h
-rw-r--r--    1 root     root          2471 Aug 13 00:23 grovelinefinder.h
-rw-r--r--    1 root     root          2253 Aug 13 00:23 groveloudness.h
-rw-r--r--    1 root     root          5590 Aug 13 00:23 grovemd.h
-rw-r--r--    1 root     root          2430 Aug 13 00:23 grovemoisture.h
-rw-r--r--    1 root     root          2115 Aug 13 00:23 groveo2.h
-rw-r--r--    1 root     root          5305 Aug 13 00:23 grovescam.h
-rw-r--r--    1 root     root          3175 Aug 13 00:23 grovespeaker.h
-rw-r--r--    1 root     root          2835 Aug 13 00:23 grovevdiv.h
-rw-r--r--    1 root     root          2135 Aug 13 00:23 grovewater.h
-rw-r--r--    1 root     root          3870 Aug 13 00:23 grovewfs.h
-rw-r--r--    1 root     root          2275 Aug 13 00:23 guvas12d.h
-rw-r--r--    1 root     root         16875 Aug 13 00:23 h3lis331dl.h
-rw-r--r--    1 root     root          7067 May 19 16:28 hal_aci_tl.h
-rw-r--r--    1 root     root          3200 Aug 13 00:23 hcsr04.h
-rw-r--r--    1 root     root          4185 Aug 13 00:23 hm11.h
-rw-r--r--    1 root     root          3081 Aug 13 00:23 hmc5883l.h
-rw-r--r--    1 root     root          7601 Aug 13 00:23 hmtrp.h
-rw-r--r--    1 root     root         10892 Aug 13 00:23 hp20x.h
-rw-r--r--    1 root     root          2765 Aug 13 00:23 ht9170.h
-rw-r--r--    1 root     root          5847 Aug 13 00:23 htu21d.h
-rw-r--r--    1 root     root          5015 Aug 13 00:23 hx711.h
-rw-r--r--    1 root     root          2311 Aug 13 00:23 ina132.h
-rw-r--r--    1 root     root          2581 Aug 13 00:23 isd1820.h
-rw-r--r--    1 root     root          3567 Aug 13 00:23 itg3200.h
-rw-r--r--    1 root     root          3132 Aug 13 00:23 jhd1313m1.h
-rw-r--r--    1 root     root          2904 Aug 13 00:23 joystick12.h
-rw-r--r--    1 root     root          5299 Aug 13 00:23 l298.h
-rw-r--r--    1 root     root          1751 Aug 13 00:23 lcd.h
-rw-r--r--    1 root     root          6700 Aug 13 00:23 lcm1602.h
-rw-r--r--    1 root     root          2479 Aug 13 00:23 ldt0028.h
-rw-r--r--    1 root     root         25248 May 19 16:28 lib_aci.h
-rw-r--r--    1 root     root          2774 Aug 13 00:23 lol.h
-rw-r--r--    1 root     root          3341 Aug 13 00:23 lpd8806.h
-rw-r--r--    1 root     root          4918 Aug 13 00:23 lsm303.h
-rw-r--r--    1 root     root         43005 Aug 13 00:23 lsm9ds0.h
-rw-r--r--    1 root     root          8727 Aug 13 00:23 m24lr64e.h
-rw-r--r--    1 root     root          3388 Aug 13 00:23 max31723.h
-rw-r--r--    1 root     root          2576 Aug 13 00:23 max31855.h
-rw-r--r--    1 root     root          4617 Aug 13 00:23 max44000.h
-rw-r--r--    1 root     root          3176 Aug 13 00:23 max5487.h
-rw-r--r--    1 root     root          3826 Aug 13 00:23 maxds3231m.h
-rw-r--r--    1 root     root          3126 Aug 13 00:23 maxsonarez.h
-rw-r--r--    1 root     root          4420 Aug 13 00:23 mhz16.h
-rw-r--r--    1 root     root          3258 Aug 13 00:23 mic.h
-rw-r--r--    1 root     root          3702 Aug 13 00:23 mlx90614.h
-rw-r--r--    1 root     root          7671 Aug 13 00:23 mma7455.h
-rw-r--r--    1 root     root          9212 Aug 13 00:23 mma7660.h
-rw-r--r--    1 root     root          7939 Aug 13 00:23 mpl3115a2.h
-rw-r--r--    1 root     root          3401 Aug 13 00:23 mpr121.h
-rw-r--r--    1 root     root         29074 Aug 13 00:23 mpu60x0.h
-rw-r--r--    1 root     root          3577 Aug 13 00:23 mpu9150.h
-rw-r--r--    1 root     root          2351 Aug 13 00:23 mq2.h
-rw-r--r--    1 root     root          2319 Aug 13 00:23 mq3.h
-rw-r--r--    1 root     root          2743 Aug 13 00:23 mq303a.h
-rw-r--r--    1 root     root          2298 Aug 13 00:23 mq5.h
-rw-r--r--    1 root     root          2278 Aug 13 00:23 mq9.h
-rw-r--r--    1 root     root          2935 Aug 13 00:23 my9221.h
-rw-r--r--    1 root     root         11104 Aug 13 00:23 nrf24l01.h
-rw-r--r--    1 root     root          2365 Aug 13 00:23 nrf8001.h
-rw-r--r--    1 root     root          4101 Aug 13 00:23 nunchuck.h
-rw-r--r--    1 root     root          4932 Aug 13 00:23 otp538u.h
-rw-r--r--    1 root     root         11302 Aug 13 00:23 pca9685.h
-rw-r--r--    1 root     root         17279 Aug 13 00:23 pn532.h
-rw-r--r--    1 root     root          2630 Aug 13 00:23 ppd42ns.h
-rw-r--r--    1 root     root          3581 Aug 13 00:23 pulsensor.h
-rw-r--r--    1 root     root          2444 Aug 13 00:23 rfr359f.h
-rw-r--r--    1 root     root          4293 Aug 13 00:23 rgbringcoder.h
-rw-r--r--    1 root     root          3132 Aug 13 00:23 rotaryencoder.h
-rw-r--r--    1 root     root          3169 Aug 13 00:23 rpr220.h
-rw-r--r--    1 root     root          2963 Aug 13 00:23 sainsmartks.h
-rw-r--r--    1 root     root          4921 Aug 13 00:23 servo.h
-rw-r--r--    1 root     root         10325 Aug 13 00:23 si114x.h
-rw-r--r--    1 root     root          5624 Aug 13 00:23 sm130.h
-rw-r--r--    1 root     root          9711 Jun  2 00:28 ssd.h
-rw-r--r--    1 root     root          3622 Aug 13 00:23 ssd1308.h
-rw-r--r--    1 root     root          3860 Aug 13 00:23 ssd1327.h
-rw-r--r--    1 root     root         22748 Aug 13 00:23 st7735.h
-rw-r--r--    1 root     root          3385 Aug 13 00:23 stepmotor.h
-rw-r--r--    1 root     root          2447 Aug 13 00:23 sx6119.h
-rw-r--r--    1 root     root          2983 Aug 13 00:23 ta12200.h
-rw-r--r--    1 root     root          5345 Aug 13 00:23 tcs3414cs.h
-rw-r--r--    1 root     root          3235 Aug 13 00:23 th02.h
-rw-r--r--    1 root     root          4657 Aug 13 00:23 tm1637.h
-rw-r--r--    1 root     root          3045 Aug 13 00:23 tp401.h
-rw-r--r--    1 root     root          6078 Aug 13 00:23 tsl2561.h
-rw-r--r--    1 root     root          2825 Aug 13 00:23 ttp223.h
-rw-r--r--    1 root     root          3633 Aug 13 00:23 ublox6.h
-rw-r--r--    1 root     root          4153 Aug 13 00:23 uln200xa.h
-rw-r--r--    1 root     root          2163 Aug 13 00:23 waterlevel.h
-rw-r--r--    1 root     root          9438 Aug 13 00:23 wt5001.h
-rw-r--r--    1 root     root          2343 Aug 13 00:23 yg1006.h
-rw-r--r--    1 root     root         11332 Aug 13 00:23 zfm20.h```



Compiling a UPM  gyroscope example from command line

    g++ -lmraa -lupm-lsm9ds0 -I/usr/include/upm/ lsm9ds0.cpp -o upmTest