# AIRO-LAB Drone Standardization
## Aoligei-C, Aoligei-V, Aoligei-L

This documentation serves as the tutorial for setting up (from hardware to software) drone **Aoligei**, including -C (for control), -V (for visual), -L (for LiDAR).


## 1. What to buy.
- Follow this [file](/documents/buy_hardware.docx) to buy all the things your need. 
- Use the files [here]() to order 3D models. 



## 2. VIM4 - the on board computer
This document provides detailed guidance of configuring VIM4. We assume that you have
- A VIM4
- A heat sink for VIM4
- A fan for VIM4
- A 512G SD card
- A stable WIFI connection
- A HDMI monitor
- A set of external keyboard and mouse

### 2.1. Install Ubuntu 22.04 to VIM4
1.  Install the heatsink and the fan to VIM4 by following the manual provided in the heatsink package. VIM4 does not work without a heatsink.

2. Insert the SD card into a SD card reader.

3. Download the software from https://docs.khadas.com/products/sbc/vim4/install-os/install-os-into-external-storage, and download “ETCHER FOR WINDOWS (X86|X64) (INSTALLER)”.

4. Download the system image from https://dl.khadas.com/products/vim4/firmware/ubuntu/generic/. Choose “vim4-ubuntu-22.04-gnome-linux-5.15-fenix-1.6-231229.img.xz”.

5. Use the software “balenaEtcher” and the system image downloaded in Step2 to install ubuntu for VIM4.

### 2.2. Post Installation
1. First setup a new username with [NATO-alphabets](https://en.wikipedia.org/wiki/NATO_phonetic_alphabet) (for easy management lah...). Make it administrator to have ```sudo``` power. 
2. Then, reboot, login with new user, and delete the ```khadas``` user. Run the following to change the username:
    ```
    sudo passwd {username}
    ```
3. For convenience, you'd better install Firefox to your VIM4 because Google Chrome is not supported.

`However, YOU MUST NOT USE "snap" to install Firefox since there might be a bug in arm-ubuntu22.04. 
 Therefore, either install Firefox with traditional "apt-get" or do not use Firefox !!`

4. Install docker in VIM4. Follow this [page](https://docs.docker.com/engine/install/ubuntu/) and it should get the task done. Remember to read everything when you are doing the installation, please do not just blindly copy every single command into your terminal.
5. Do the following to get your Docker image:
    ```
    git clone https://github.com/HKPolyU-UAV/airo_docker_lib
    ./build_hehe.sh vimswift
    ```
   After building the image, do:
   ```
   ./run_hehe.sh vimswift
   ```
6. Voila! You have an usable programming environment now! When modifying your code, we suggest you to use VScode on your own laptop and connect the drone with ```ssh```. You can refer to this [documentation](https://github.com/pattylo/useful_tools/blob/main/vscode_github/vscode_github.md) for more info.

## 3. Firmware Setup
- A computer w/ Ubuntu >=20.04
- A Kakute FCU (v1.3 || v2)
- A Type-C USB wire
- A 4G SD card (if you are using Kakute v1.3)

### 3.1. Firmware Upgrade
1. Install DFU library on your own computer
    ```
    sudo apt install dfu-util -y
    ```
2. Erase and load the binary file to the FCU.
    ```
    git clone https://github.com/HKPolyU-UAV/Aoligei.git && cd Aoligei
    ```
    Then, unplug your USB connection, press the DUF button, plug your FCU into your computer, and unpress the DFU button. Then, do:
    ```
    dfu-util -a 0 --dfuse-address 0x08000000:force:mass-erase:leave -D ./build/holybro_kakuteh7_bootloader.bin 
    # ignore the error
    # use "holybro_kakuteh7v2_bootloader" if your are using v2
    ```
    And then, unplug your USB connection, press the DUF button, plug your FCU into your computer, and unpress the DFU button. Now do:
    ```
    dfu-util -a 0 --dfuse-address 0x08000000 -D ./build/holybro_kakuteh7_bootloader.bin
    # again, use "holybro_kakuteh7v2_bootloader" if your are using v2
    ```
3. Load firmware to the FCU. Open QGC, and go to "Firmware" to upload ```.px4``` to FCU. Note that you should tick on "advance settings", and then select "Custom firmware file..." under the drop-down options.

   <div align=center>
   <img src="/documents/figures/custom_firmware.png"/>
   </div>

    Click "OK", it should pop out a file selection panel. Go the "build" file of this repo, and select [this (v1.3)](/build/holybro_kakuteh7_default.px4) or [that (v2)](/build/holybro_kakuteh7v2_default.px4).

4. If you want to build from source on your own, refer to [this](./documents/fcu.md)

## 4. Pre-configuration of the FCU
### 4.1. Parameter settings
We assume you have at least finished the procedure "C" aforementioned. This part provides some pre-configurations of the FCU. Therefore, we assume you have

- An assembled drone with a FCU, a VIM4, four motors, a battery. (If you still don't know how to install ubuntu22 or docker file into VIM4, never mind. It won't affect this step.)
- QGC in Ubuntu. (Windows would be fine, but Ubuntu is preferred.)
- A type-C cable.

**NOTING: This drone is currently only used in a motion-capture-located in-door room. Out door flight is currently not supported.**

You may need half a minute to connect the FCU with QGC. When the connection is stable, you can see a content on the left of GQC.
Select "Parameters", and the following parameters need to be reset:

|    Parameter     |  Value   |                 Comment                 |
|:----------------:|:--------:|:---------------------------------------:|
|   SYS_HAS_MAG    |    0     |  This FCU does not have a magnetometer  |
|   SYS_HAS_GPS    | Disabled |     This drone does not have a GPS      |
|   SYS_HAS_BARO   | Disabled |   This FCU does not have a barometer    |
| SYS_MC_EST_GROUP |   ekf2   |                                         |
|    SDLOG_MODE    | disabled | Disable the log file to save CPU memory |

Reboot the FCU with QGC, when the FCU is reloaded, set

| Parameter      |  Value   | Comment   |
|:--------------:|:--------:|:---------:|
|  EKF2_EV_CTRL  |    15    |           |
|  EKF2_HGT_REF  |  Vision  |           |
|  MAV_0_CONFIG  | Disabled |           |
|  MAV_1_CONFIG  |  TELEM1  |           |
| SER_TEL1_BAUD  |  921600  |           |

Reboot the FCU with QGC, and this step is completed.



### 4.2. Quadrotor calibration
Then, we start to calibrate the quadrotor. Noting that different people have different styles.
Therefore, you don't need to follow EVERYTHING. We assume you can connect the FCU with QGC.

#### 4.2.1 Sensors
Calibrate Gyroscope, Accelerator, Level Horizon, and Orientations, respectively.
The "Autopilot Orientation" is "ROTATION_NONE" (default.)

#### 4.2.2. Radio
We need to connect the FASST receiver to the radio. Firstly, open the radio and switch it to FASST mode.
Secondly, push the mode select button on the receiver and then power on the receiver.
Now the light on the receiver is shinning and remains green. Then connection is finished.
We use "Mode2" here, as shown bellow. (If you have your own style, follow you own style.)

<div align=center>
<img src="/documents/figures/mode2.png"/>
</div>

Then, calibrate the radio as others do. Nothing special; therefore, detailed description is omitted here.

#### 4.2.3. Flight mode setting
The following is the mode setting of our group. You can have your own style.

<div align=center>
<img src="/documents/figures/modesetting.png">
</div>

#### 4.2.4. Power setup and ESC calibration
**DO NOT INSTALL PROPELLERS!!**

Connect the drone with the battery. Push the first "Calculate" in the following figure.
Enter the measured voltage of the battery into the blank.
Then, and write the series of your battery (3, 4, or 6) into "Number of Cells"

<div align=center>
<img src="/documents/figures/batterycali.png">
</div>

Then, calibrate the ESC. Click the button in the following figure. Just follow the instructions.
<div align=center>
<img src="/documents/figures/ESCcali.png">
</div>

#### 4.2.5. Calibrate Actuators
Firstly, you should see a screen like this.

<div align=center>
<img src="/documents/figures/actuators_cali.png" width="480">
</div>

See the right column, set the MAIN1-2 and MAIN3-4 to "DShot1200", and then press "Identify & Assign Motors".

Secondly, see the left column, the positions (X and Y) from Motor 1 to 4 are shown below.

|  Motor   |  Position X  |  Position Y  |  Direction CCW  |
|:--------:|:------------:|:------------:|:---------------:|
|  Motor1  |     0.08     |     0.08     |       Yes       |
|  Motor2  |    -0.08     |    -0.08     |       Yes       |
|  Motor3  |     0.08     |    -0.08     |       No        |
|  Motor4  |    -0.08     |     0.08     |       No        |

Noting that the "0.08" here is especially designed for our own drone.
You'd better use a ruler to measure the semi-major axis of your own drone.
Then, the expected rotation direction of the four motors are shown in the QGC.

<div align=center>
<img src="/documents/figures/uav.png" width="350">
</div>

Finally, you need to reverse the rotation direction if some motors don't rotate as expected.
Use the buttons "Set Spin Direction 1" and "Set Spin Direction 2" to correct them, which are shown below.

<div align=center>
<img src="/documents/figures/motor_cali.png" width="350">
</div>

### 4.3 Post Calibration
Unplug your SD card, connect the SD card to your PC. Create a new folder "etc" to the SD card.
create "extras.txt" in "etc" folder. Then, copy the following items into the txt file.

```
mavlink stream -d /dev/ttyS0 -s LOCAL_POSITION_NED -r 100
mavlink stream -d /dev/ttyS0 -s ATTITUDE_TARGET -r 100
mavlink stream -d /dev/ttyS0 -s BATTERY_STATUS -r 100
```

Congratulations! If everything goes well, you have finished pre-configurations of the drone.

`Tips: Although we provide detailed calibration instruction here,
many users (including ourselves) would meet many confused bugs while doing the step.
That is because different people have totally different styles, use different hardware, have different QGC version, have different radios, et al. 
PLease feel free to ask the "Maintainers" shown below and share us your weired bugs.
We might don't know how to fix your bugs, but we are curious about them. Thank you very much!`


## 5. System Identification 
@ https://github.com/RockyJBL
Please follow the [link](https://github.com/HKPolyU-UAV/airo_control_interface.git) here to conduct the system identification.

## Maintainers
[Yefeng](https://github.com/Yang-Yefeng) @ AIRO-LAB @ RCUAS, HKPolyU <br>
[RockyJBL](https://github.com/RockyJBL) @ AIRO-LAB @ RCUAS, HKPolyU <br>
[VITO](https://github.com/Vitollee) @ AIRO-LAB @ RCUAS, HKPolyU <br>
[pattylo](https://github.com/pattylo) @ AIRO-LAB @ RCUAS, HKPolyU

<!-- # PX4 Drone Autopilot

[![Releases](https://img.shields.io/github/release/PX4/PX4-Autopilot.svg)](https://github.com/PX4/PX4-Autopilot/releases) [![DOI](https://zenodo.org/badge/22634/PX4/PX4-Autopilot.svg)](https://zenodo.org/badge/latestdoi/22634/PX4/PX4-Autopilot)

[![Nuttx Targets](https://github.com/PX4/PX4-Autopilot/workflows/Nuttx%20Targets/badge.svg)](https://github.com/PX4/PX4-Autopilot/actions?query=workflow%3A%22Nuttx+Targets%22?branch=master) [![SITL Tests](https://github.com/PX4/PX4-Autopilot/workflows/SITL%20Tests/badge.svg?branch=master)](https://github.com/PX4/PX4-Autopilot/actions?query=workflow%3A%22SITL+Tests%22)

[![Discord Shield](https://discordapp.com/api/guilds/1022170275984457759/widget.png?style=shield)](https://discord.gg/dronecode)

This repository holds the [PX4](http://px4.io) flight control solution for drones, with the main applications located in the [src/modules](https://github.com/PX4/PX4-Autopilot/tree/main/src/modules) directory. It also contains the PX4 Drone Middleware Platform, which provides drivers and middleware to run drones.

PX4 is highly portable, OS-independent and supports Linux, NuttX and MacOS out of the box.

* Official Website: http://px4.io (License: BSD 3-clause, [LICENSE](https://github.com/PX4/PX4-Autopilot/blob/main/LICENSE))
* [Supported airframes](https://docs.px4.io/main/en/airframes/airframe_reference.html) ([portfolio](https://px4.io/ecosystem/commercial-systems/)):
  * [Multicopters](https://docs.px4.io/main/en/frames_multicopter/)
  * [Fixed wing](https://docs.px4.io/main/en/frames_plane/)
  * [VTOL](https://docs.px4.io/main/en/frames_vtol/)
  * [Autogyro](https://docs.px4.io/main/en/frames_autogyro/)
  * [Rover](https://docs.px4.io/main/en/frames_rover/)
  * many more experimental types (Blimps, Boats, Submarines, High altitud Pe balloons, etc)
* Releases: [Downloads](https://github.com/PX4/PX4-Autopilot/releases)


## Building a PX4 based drone, rover, boat or robot

The [PX4 User Guide](https://docs.px4.io/main/en/) explains how to assemble [supported vehicles](https://docs.px4.io/main/en/airframes/airframe_reference.html) and fly drones with PX4.
See the [forum and chat](https://docs.px4.io/main/en/#getting-help) if you need help!


## Changing code and contributing

This [Developer Guide](https://docs.px4.io/main/en/development/development.html) is for software developers who want to modify the flight stack and middleware (e.g. to add new flight modes), hardware integrators who want to support new flight controller boards and peripherals, and anyone who wants to get PX4 working on a new (unsupported) airframe/vehicle.

Developers should read the [Guide for Contributions](https://docs.px4.io/main/en/contribute/).
See the [forum and chat](https://docs.px4.io/main/en/#getting-help) if you need help!


### Weekly Dev Call

The PX4 Dev Team syncs up on a [weekly dev call](https://docs.px4.io/main/en/contribute/).

> **Note** The dev call is open to all interested developers (not just the core dev team). This is a great opportunity to meet the team and contribute to the ongoing development of the platform. It includes a QA session for newcomers. All regular calls are listed in the [Dronecode calendar](https://www.dronecode.org/calendar/).


## Maintenance Team

Note: This is the source of truth for the active maintainers of PX4 ecosystem.

| Sector | Maintainer |
|---|---|
| Founder | [Lorenz Meier](https://github.com/LorenzMeier) |
| Architecture | [Daniel Agar](https://github.com/dagar) / [Beat Küng](https://github.com/bkueng)|
| State Estimation | [Mathieu Bresciani](https://github.com/bresch) / [Paul Riseborough](https://github.com/priseborough) |
| OS/NuttX | [David Sidrane](https://github.com/davids5) |
| Drivers | [Daniel Agar](https://github.com/dagar) |
| Simulation | [Jaeyoung Lim](https://github.com/Jaeyoung-Lim) |
| ROS2 | [Beniamino Pozzan](https://github.com/beniaminopozzan) |
| Community QnA Call | [Ramon Roche](https://github.com/mrpollo) |
| [Documentation](https://docs.px4.io/main/en/) | [Hamish Willee](https://github.com/hamishwillee) |

| Vehicle Type | Maintainer |
|---|---|
| Multirotor | [Matthias Grob](https://github.com/MaEtUgR) |
| Fixed Wing | [Thomas Stastny](https://github.com/tstastny) |
| Hybrid VTOL | [Silvan Fuhrer](https://github.com/sfuhrer) |
| Boat | x |
| Rover | x |

See also [maintainers list](https://px4.io/community/maintainers/) (px4.io) and the [contributors list](https://github.com/PX4/PX4-Autopilot/graphs/contributors) (Github). However it may be not up to date.

## Supported Hardware

Pixhawk standard boards and proprietary boards are shown below (discontinued boards aren't listed).

For the most up to date information, please visit [PX4 user Guide > Autopilot Hardware](https://docs.px4.io/main/en/flight_controller/).

### Pixhawk Standard Boards

These boards fully comply with Pixhawk Standard, and are maintained by the PX4-Autopilot maintainers and Dronecode team

* FMUv6X and FMUv6C
  * [CUAV Pixahwk V6X (FMUv6X)](https://docs.px4.io/main/en/flight_controller/cuav_pixhawk_v6x.html)
  * [Holybro Pixhawk 6X (FMUv6X)](https://docs.px4.io/main/en/flight_controller/pixhawk6x.html)
  * [Holybro Pixhawk 6C (FMUv6C)](https://docs.px4.io/main/en/flight_controller/pixhawk6c.html)
  * [Holybro Pix32 v6 (FMUv6C)](https://docs.px4.io/main/en/flight_controller/holybro_pix32_v6.html)
* FMUv5 and FMUv5X (STM32F7, 2019/20)
  * [Pixhawk 4 (FMUv5)](https://docs.px4.io/main/en/flight_controller/pixhawk4.html)
  * [Pixhawk 4 mini (FMUv5)](https://docs.px4.io/main/en/flight_controller/pixhawk4_mini.html)
  * [CUAV V5+ (FMUv5)](https://docs.px4.io/main/en/flight_controller/cuav_v5_plus.html)
  * [CUAV V5 nano (FMUv5)](https://docs.px4.io/main/en/flight_controller/cuav_v5_nano.html)
  * [Auterion Skynode (FMUv5X)](https://docs.auterion.com/avionics/skynode)
* FMUv4 (STM32F4, 2015)
  * [Pixracer](https://docs.px4.io/main/en/flight_controller/pixracer.html)
  * [Pixhawk 3 Pro](https://docs.px4.io/main/en/flight_controller/pixhawk3_pro.html)
* FMUv3 (STM32F4, 2014)
  * [Pixhawk 2](https://docs.px4.io/main/en/flight_controller/pixhawk-2.html)
  * [Pixhawk Mini](https://docs.px4.io/main/en/flight_controller/pixhawk_mini.html)
  * [CUAV Pixhack v3](https://docs.px4.io/main/en/flight_controller/pixhack_v3.html)
* FMUv2 (STM32F4, 2013)
  * [Pixhawk](https://docs.px4.io/main/en/flight_controller/pixhawk.html)

### Manufacturer supported

These boards are maintained to be compatible with PX4-Autopilot by the Manufacturers.

* [ARK Electronics ARKV6X](https://docs.px4.io/main/en/flight_controller/arkv6x.html)
* [CubePilot Cube Orange+](https://docs.px4.io/main/en/flight_controller/cubepilot_cube_orangeplus.html)
* [CubePilot Cube Orange](https://docs.px4.io/main/en/flight_controller/cubepilot_cube_orange.html)
* [CubePilot Cube Yellow](https://docs.px4.io/main/en/flight_controller/cubepilot_cube_yellow.html)
* [Holybro Durandal](https://docs.px4.io/main/en/flight_controller/durandal.html)
* [Airmind MindPX V2.8](http://www.mindpx.net/assets/accessories/UserGuide_MindPX.pdf)
* [Airmind MindRacer V1.2](http://mindpx.net/assets/accessories/mindracer_user_guide_v1.2.pdf)
* [Holybro Kakute F7](https://docs.px4.io/main/en/flight_controller/kakutef7.html)

### Community supported

These boards don't fully comply industry standards, and thus is solely maintained by the PX4 public community members.

### Experimental

These boards are nor maintained by PX4 team nor Manufacturer, and is not guaranteed to be compatible with up to date PX4 releases.

* [Raspberry PI with Navio 2](https://docs.px4.io/main/en/flight_controller/raspberry_pi_navio2.html)
* [Bitcraze Crazyflie 2.0](https://docs.px4.io/main/en/complete_vehicles/crazyflie2.html)

## Project Roadmap

**Note: Outdated**

A high level project roadmap is available [here](https://github.com/orgs/PX4/projects/25).

## Project Governance

The PX4 Autopilot project including all of its trademarks is hosted under [Dronecode](https://www.dronecode.org/), part of the Linux Foundation.

<a href="https://www.dronecode.org/" style="padding:20px" ><img src="https://mavlink.io/assets/site/logo_dronecode.png" alt="Dronecode Logo" width="110px"/></a>
<a href="https://www.linuxfoundation.org/projects" style="padding:20px;"><img src="https://mavlink.io/assets/site/logo_linux_foundation.png" alt="Linux Foundation Logo" width="80px" /></a>
<div style="padding:10px">&nbsp;</div> -->
