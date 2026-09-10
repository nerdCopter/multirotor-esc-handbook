# FAQ / ESC tuning resources

#### Videos
* BLHeli32
  * JB/Harrell full series: https://www.youtube.com/playlist?list=PLwoDb7WF6c8kXOyPdBog1wtRcxnXMasUb
  * JB/Harrell desync specific: https://youtu.be/oKcyXR7Yx64 (@21:17)
  * Pawel Spychalski Demag-Compensation: https://youtu.be/c94e9TCCP8Y
  * Kabab micro rampup: https://youtu.be/obObn1oGWmU <-- read Mr.ShutterBug's comment as well!
  * JB's variable PWM https://youtu.be/xuQeJA4EGr8
  * Chris Rosser BLHeli32 2021: https://youtu.be/7WeHTb7aBrE
  * Chris Rosser BLHeli32 2024: https://youtu.be/6gv0_jTEYZM
* BlueJay
  * JB's Bluejay vid: https://youtu.be/yEDhnBUFQNI
  * Chris Rosser BlueJay 2024: https://youtu.be/EhYKeZfSQIw (Caveat: Bluejay devs seem to recommend 48khz, not 96khz, and also recommend do ***not*** set startup powers to max)
* AM32
  * JB's 32bit AM32 vid: https://youtu.be/yOeVj6P9PSU
  * AM32 Settings Explained playlist by "High Energy Failures": https://www.youtube.com/playlist?list=PLrxUiYCo7HwrIz1uE0aIfNT4GUgVMwc1t
  * Chris Rosser AM32 2024: https://youtu.be/3SHzyUaypFw

#### Heat
* As MOSFET switching frequency increases, so does heat generation.
* Some reports exist of too hot ESC's/Motors at 128khz PWM depending on the brand/model of the ESC/Motors.

#### Desyncs
* Desyncs are typically a factor of ESC settings.
* Set `Motor Timing` to a some static value of personal preference.  23 is good for freestyle.  Racers typically use higher values.
* `Demag Compensation` also a factor but try static timing first, and increase Demag-Comp only as much as needed to resolve.

#### Racing Considerations
* Demag compensation low saves roughly 7K RPM (20%) on high kv motors. Although off versus low has minimal effect, some prefer OFF. Thanks to @FreedomDuck for this information. *(Ultralight 6S 2100kv to 2150kv.)*

#### BLHeli32 (32-bit)
* https://github.com/bitdump/BLHeli/releases
* Some BLHeli_32 ESC's can be flashed with with other firmware, but requires STLink or similar devices and bootloader unlocking. Search for AM32 and/or Escape32 resources on the subject.

#### BlueJay (8-bit)
* Discord: https://discord.gg/ddyzguPB5t
* https://github.com/bird-sanctuary/bluejay/releases
* Bluejay startup min/max, etc. https://github.com/bird-sanctuary/bluejay/wiki/Setup
* Bluejay config migration: https://github.com/bird-sanctuary/bluejay/wiki/Migrating-from-BLHeli_S
* Configurator: https://esc-configurator.com/
* 24khz=responsive; 96khz=extended-flight-time; 48khz is a balance.
* Some ESC/motors have issue starting up at 96. Bluejay devs recommend 48  and was considering removal of 96.
* whoop's static idle: 9%-14% recommended.

#### AM32 (32-bit)
* Discord: https://discord.gg/h7ddYMmEVV
* https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware/releases
* Configurator: https://am32.ca/
* Wiki: https://wiki.am32.ca/
  * OLD wiki here: https://github.com/AlkaMotors/AM32-MultiRotor-ESC-firmware/wiki
* We are all aware of PWM fixed and PWM Variable (Low/High), but now AM32 adds a "By RPM".

#### Escape32 (32-bit)
* Discord: https://discord.gg/aed6xdSM5Y
* https://github.com/neoxic/ESCape32

