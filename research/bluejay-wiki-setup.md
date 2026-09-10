This page aims to help you set up specific Bluejay features.


# Startup Power and Motor Idle Setup
> **Perform a hardware inspection before proceeding (check solder joints, wires, motor bolts. bearings etc)**

It is important to set the min/max startup limits in ESC-Configurator and the motor idle setting in your flight controller correctly to protect your motors and ESCs during the jammings produced in crashes and turtling. 

If you are having problems with your motors not spinning then read the sections below.

## Min/Max Startup Limits
>The ESC cannot tell whether it is outputting too much current when the motor is stalled because there is no phase current feedback in any BLHeli_S based ESC. This is 'open loop' control. 

>The worst case scenario of this are **burnt motors/ESCs etc**. 

>Setting the **lowest possible startup limits** is the only thing that reduces the risk of this occurring.

The startup limits represent a fixed lower throttle value Bluejay outputs on initial startup before using the value given by the FC. The current given by the initial startup value varies depending on the equipment. 

First Bluejay tries to start motors at min startup setting. If this fails it incrementally increases the startup power up each startup attempt to the max startup setting. 

Should the motors not spin up when max startup power is applied, the motors are assumed to be stalled.

## Motor Idle

Motor Idle also known as Digital Idle is a setting implemented in the flight controller firmware. This setting is very important, because as soon as the ESC realizes that motor is spinning it delegates the throttle authority to the flight controller, so if this setting is low the **motor will stall and stop after being spooled up**.

## Setting Min/Max Startup Limits and Motor Idle Settings
> For all values: Remember the goal is to **find lowest values to give the maximum risk reduction**.

Use the values in the [[Recommended Settings|Setup#recommended-settings]] tables as a starting point. Arm the quad and observe the propeller behaviour.

Observed propeller behaviour:
* They spin and stop repeatedly - Increase motor idle by 3%. Check that dynamic idle is disabled.
* Do not spin or weakly spin and stop - Increase both min/max startup power by 10.
Repeat until motors start or they start and stop continuously.
* Consistently spin on the initial values: 
Try reducing min/max startup power by 10 until they no longer start consistently and then repeat the above.
* Start and then stall in flight - It could potentially happen when motor idle is really low, so set motor idle to a convenient value.
* Fail to start when battery is low:
Increase motor idle or max startup maximum startup power by 10.

### Major difference from startup limits listed in [[Recommended Settings|Setup#recommended-settings]] tables
As each setup is different, some variation may be observed. However too much variation can indicate a problem. 

For any difference greater than +/- 10 from a similar quad in the [[Recommended Settings|Setup#recommended-settings]] tables:
> **Perform a detailed hardware inspection before proceeding (check solder joints, wires, motor bolts. bearings etc)**

If it is a close match to an entry above, please report any increases or decreases to the team on Discord or as a Github issue. Please include a screenshot of your ESC settings from ESC-Configurator, and a `diff all` from your flight controller. This may help catch an issue early before it does damage to your ESC or another users.

If there is no close match, proceed with caution. For a quad with a deadtime <25, any adjustment outside of +/- 10 from the most similar entry is likely too much and will reduce your startup protection. Make sure you have followed the steps around motor idle, and consider reporting as above.

* Testing with crashed quads has shown they normally do not require an increase in the startup limits.

* Lower KV motor/prop combo may require increased startup power at 1S/2S as demonstrated in the table below

## Recommended settings

In this section we are going to give values that have worked for the testing team to give you the maximum protection of your motors and ESCs. 
> **If you use larger values you increase the risk of burning your motors and ESCs.** 

> These values will help increase the protection of the motors, but external events like crashings and jamming always affect to the durability of your hardware.

> Worn batteries may affect to the necessary settings in your craft. 

These settings use static idle for maximum reliability. Dynamic idle can offer improved handling in several scenarios, but it can introduce extra risk, please review the [[Dynamic Idle|Setup#dynamic-idle]] section

If you want the maximum possible protection, you still need to confirm these are the lowest realistic values for your particular type of quad - see [[Setting Min/Max Startup Limits and Motor Idle Settings | Setup#setting-minmax-startup-limits-and-motor-idle-settings]]

### Motor Voltage above 2S
| Motor Voltage                      | min/max startup power | Motor Idle |
|------------------------------------|-----------------------|------------|
| 4S and above                       | 1010/1020             | 5,5        |
| 3S                                 | 1020/1040             | 5,5        |

### Motor Voltage at or below 2S
For voltages at 1/2S, please find the deadtime of your ESC - one way to do this is looking at the layout inside ESC-Configurator, with a format x_y_nn, where nn is the deadtime.

>**Proceed with caution!** It is recommended to check whether you can lower the limits for your particular quad, see [[Setting Min/Max Startup Limits and Motor Idle Settings | Setup#setting-minmax-startup-limits-and-motor-idle-settings]]




> **Avoid dynamic idle on 1S and 2S prior to Betaflight 4.5!**
> Prior to Betaflight 4.5, users with low voltage 1S/2S craft and dynamic idle would find it is harder to start the motors, and mistakenly crank up min/max startup power **inadvertently disabling motor protection**. In that case the risk of damaging the ESC is very high. 

> With Betaflight 4.5, a parameter was introduced that should alleviate this concern, see https://github.com/betaflight/betaflight/pull/12432
It is suggested to use the motor idle value provided as the dynamic idle start value.

> Dynamic idle can offer improved handling in several scenarios, but it can introduce extra risk, see  [[Dynamic Idle|Setup#dynamic-idle]] section for further information

A layout of O_H_5 has a deadtime of 5 for example. 
#### Deadtime <30
| Motor Voltage                      | min/max startup power | Motor Idle |
|------------------------------------|-----------------------|------------|
| 48khz 1S 0802 19500kv deadtime 5   | 1080/1120             | 9          |
| 48/96khz 1S 1102 22000kv deadtime 5   | 1020/1040            | 8      |
| 48khz 1S 0702 28500kv deadtime 5   | 1040/1060             | 8          |
| 48khz 1S 0702 32500kv deadtime 10  | 1050/1070             | 8          |
| 48khz 1S 0603 17000kv deadtime 20  | 1090/1150             | 10         |

#### Deadtime >=50
Deadtimes starting around 50 can require a drastic increase in startup limits when kwad is powered at 1S and 2S. 

Try one of the values from the table above **first**, **especially if you have an ESC near 50**. If they do not work, or your is significantly greater than 30, consider using the values below

> Using these settings on a low deadtime ESC or above 1S/2S **will result in no motor protection**

| Motor Voltage                      | min/max startup power | Motor Idle |
|------------------------------------|-----------------------|------------|
| 48khz 1S 0802 19500kv deadtime 70  | 1125/1200             | 16         |

## Dynamic Idle
> **Dynamic idle introduces extra risk of ESC and/or motor failure in props blocked scenarios**

Due to the lack of current protection/feedback on BlHeli type ESCs, dynamic idle can be risky as it will continue to increase the motor speed in response to additional load when hitting objects or crashing. This means that the motors could continue to operate at or beyond full load for an extended period of time, possibly damaging the ESC or the motors, during these crash scenarios.

>**ATTENTION:** Avoid dynamic idle on 1S and 2S prior to Betaflight 4.5!

Prior to Betaflight 4.5, users with low voltage 1S/2S craft and dynamic idle would find it is harder to start the motors, and mistakenly crank up min/max startup power **inadvertently disabling motor protection**. In that case the risk of damaging the ESC is very high. 

With Betaflight 4.5, a parameter was introduced that should alleviate this concern, see https://github.com/betaflight/betaflight/pull/12432

For further information regarding dynamic idle, see the following resources: 

[Betaflight documentation on dynamic idle for tuning information](https://betaflight.com/docs/wiki/guides/current/dynamic-idle) 

[Betaflight Discord](https://discord.gg/XzM3a9HTsZ)

# Extended DSHOT Telemetry - EDT
Bluejay implements [EDT - external DSHOT telemetry](https://github.com/bird-sanctuary/extended-dshot-telemetry) on open extension of the DSHOT bi-directional telemetry functionality. This allows the ESC to send additional telemetry back to the flight controller and also allows for more advanced safety features like for example to wait for en EDT-Handshake before allowing the motors to be armed.

EDT provides a lot of additional information which will help you in debugging issues but also aid in improving your tune.

## How to use?
In order to use EDT, the flight controller firmware needs to support it. If it does, Bluejay will send the EDT frames automatically.

### Betaflight
Betaflight 4.4.x supports this feature, it needs to be activated first (unless you are in the "Motor" Tab where it is enabled by default).

> This below information can be moved into the Betaflight Wiki

#### Setup
To activate EDT in Betaflight execute the following commands in the CLI:

```
set dshot_edt=on
save
```


#### Testing

**MAKE SURE THE PROPS ARE OFF FOR THE NEXT STEPS!**

To now test it, run the following commands:
```
dshotprog 255 13
motor 255 1060
dshot_telemetry_info
```
this will yield output similar to this:

![Screenshot from 2023-03-05 00-29-34](https://user-images.githubusercontent.com/978192/222934164-bf301ded-f094-4c98-9039-6ab478729ef2.png)

The `eRPM` and `RPM` colums will have an output as long as bi-directional dshot is enabled. In the type column you will see a couple letters indicating the type of packages being received. The `TEMP` column will show the temperature measured by the internal temperature sensor of the EFM8 micro controller.

# Temperature Protection
Prior to Bluejay version 0.19 temperature protection has not been working at all. This has been fixed with version 0.19 and a couple of issues arose with this:

* Temperature protection can cause issues on some 1S builds if not configured correctly
* AIO boards with badly out of whack temp sensor calibrations have been found

**For the above reasons we have decided to disable temperature protection by default and make it an opt in feature instead.**

## Setup (Betaflight)
In order to make temperature protection work, the following steps are necessary (the description here is for Betaflight, but will be very similar for any other flight-controller firmware which implements EDT):

1. Enable ESC Temperature OSD Element in Betaflight
2. Enable EDT via CLI:

    ```
    set dshot_edt=on
    save
    ```

3. Set power rating in esc-configurator. Generally speaking this should always be 2S+, unless you have an AIO that is rated for 1S only, then it is very likely that the 1S setting is the correct one, but this is unfortunately not always the case. You will have to check this via the ESC Temperature OSD Element that you enabled in the step above.
4. Check temperature reading - if they are in a sane range, you can enable temperature protection. A sane range would be ambient temperature +10-20 degrees. Keep in mind that it is likely that not all ESCs will read the same exact value, but they should be in the same ballpark. If the temp readings are 0 or on the upper limit of 255, then something is not correct, you can try changing the power rating and checking the measurements again.


