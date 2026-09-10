## Startup Power

This table shows the correspondence between the startup power settings of BLHeli_S and Bluejay.

| BLHeli_S      | Bluejay            |                    |                    |
|---------------|--------------------|--------------------|--------------------|
| Startup Power | Min. Startup Power | Max. Startup Power | RPM Power (Rampup) |
| 0.031         |   2 (1001)         |  1 (1004)          |  2x                |
| 0.047         |   4 (1002)         |  2 (1008)          |  2x                |
| 0.063         |   6 (1003)         |  3 (1012)          |  3x                |
| 0.094         |   8 (1004)         |  4 (1016)          |  4x                |
| 0.125         |  12 (1006)         |  6 (1024)          |  5x                |
| 0.188         |  18 (1009)         |  9 (1035)          |  6x                |
| 0.25          |  24 (1012)         | 12 (1047)          |  7x                |
| 0.38          |  36 (1018)         | 18 (1071)          |  8x                |
| 0.50          |  50 (1024)         | 25 (1098)          |  9x                |
| 0.75          |  74 (1036)         | 37 (1145)          | 10x                |
| 1.00          | 100 (1049)         | 50 (1196)          | 11x                |
| 1.25          | 124 (1061)         | 62 (1243)          | 12x                |
| 1.50          | 150 (1073)         | 75 (1294)          | 13x                |

- **Minimum startup power:** Minimum power when starting motors. Increase if motors are not able to start with low throttle input.
- **Maximum startup power:** Limits power when starting motors or reversing direction.
- **RPM Power Protection (Rampup):** Limits how fast power can be increased. Lower values will avoid power spikes but can also decrease acceleration.

## PWM Frequency

In addition to 24 kHz PWM in BLHeli_S, Bluejay also allows using 48 and 96 kHz.

Increasing the PWM frequency can increase flight time when using smaller motors, but usually idle throttle also needs to be increased to reliably spin the motors.

It is best to use new clean motors when tuning the settings because dirty/used motors will need more power to the point where it could risk burning them.

In Betaflight or ESC configurator you can do a motor test to determine the throttle value needed to keep the motors spinning reliably. You probably need to raise throttle to a higher value first, in order to start the motors, and then back down to find how low you can go without stopping them. You want idle throttle to be a bit more than the lowest value that keeps them spinning.
E.g. if `1060` was the lowest value needed to start the motors, corresponding to an idle of `6.0` in Betaflight, you might need idle somewhere between `7.0` - `10.0` (`1070` - `1100`) to ensure that they do not stop.

The necessary settings depends on the motors, ESCs, battery power, PWM frequency, etc.

If the motors cannot spin up at all, you need to increase the Bluejay max startup power setting.
The min startup power setting is used to be able to start the motors at low throttle.
E.g. if a throttle of `1150` is needed to make the motors start spinning, but an idle throttle of `1100` is sufficient to keep them spinning once they are started.