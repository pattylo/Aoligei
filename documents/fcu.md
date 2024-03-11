# Firmware being built from source

1. Git clone PX4-Autopilot
    ```
    git clone https://github.com/PX4/PX4-Autopilot
    ```
2. Checkout tag version.
    ```
    git checkout tags/v1.14.0
    ```
3. Then do,
    ```
    git submodule update --init --recursive
    make holybro_kakuteh7_bootloader # || make holybro_kakuteh7v2_bootloader 
    ```
4. Do the DFU step as described [here](README.md#firmware-upgrade)
5. Now build FCU firmware.
    ```
    make holybro_kakuteh7 # || make holybro_kakuteh7v2
    ```