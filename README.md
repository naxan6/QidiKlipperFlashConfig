# Introduction

Links etc. were valid on 2021-11-17

**DISCLAIMER: be warned, here be dragons**
*it worked for me - but it could fail **you***

**summary**
This documentation explains the most important steps to install Klipper
(https://www.klipper3d.org/) on your Qidi Tech X-Plus or Qidi Tech X-Max.
Currently you will have to flash klippers firmware via an ST-Link v2 (or similar).

**what is missing**
*  maybe precompiled klipper.bin (bad idea,  it never would be up to date)
*  my docker compose definitions (+ maybe a short introduction for those that want to go down the docker way; atm still a bad idea: it's not "just working")
*  my Klipper Screen - config (didn't see an easy way to put it into a docker container so installed it directly the the raspberry)

**how to read this**

*  if you don't have a klipper stack running (or don't know what I mean
   by that), start with that, which means you would start with chapter
   0, and after that 1, 2, 3, 5
*  if you already have a klipper stack runnning, do chapters 1, 2, 3, 5
*  if you want to go back to original qidi firmware, do chapter 4 with
   the firmware-backup from chapter 3.

# 0. Install klipper/moonraker/fluidd
-----------------------------------------------------------------

*why do I need  all this stuff?*
*  **klipper** connects to the klipper-firmware-part on the mainboard
    https://github.com/Klipper3d/klipper
   * **klipper-firmware**  
     the klipper-fimrmware-part - has to be built matching for your printers chips

*  **moonraker** connects to klipper and publishes functions via web
   apis
    https://github.com/Arksine/moonraker

*  **fluidd** is a webclient that connects to moonraker and therefore
   indirectly to klipper
       (there are other clients, see KIAUH webpage e.g.)
       https://github.com/fluidd-core/fluidd
       sidenote: I tried mainsail and prefer fluidd which is said to be a fork of mainsail

* **Raspberry Pi**  
  * runs alle th stuff above  
  * should be fast enough (it said it should be at least a Raspberry Pi 2b).
  * connect the Raspberry Pi to the serial port on the Qidi X-Plus/X-Max 
  * Mainboard (I used an USB to Serial Adapter; some of raspberries own contacts should also work): 
    ```  
    RX->TX
    TX->RX
    Ground->Ground
    5V->5V
    ```
    (Sidenote: I connected 5V from Qidis Mainboard to the 5V on the USB-To-Serial-Adapter, but put a tape onto the USB 5V-Pin inside the USB plug to avoid powering the mainboard through the raspberry)
   * ![Alt text](/images/mainboard_overview.jpg?raw=true "Mainboard")
   * ![Alt text](/images/focusMainboardSerial.jpg?raw=true "Serial")

*where to start?*
  *  I for myself started with **prind** (docker compose based setup) on an Raspberry Pi OS and altered it to my wishes
   but there are many additional hurdles if you're not used to docker so I would not recommend this to everybody... it's also not quirks free at the moment (have to restart the  klipper-container to reconnect to the printer because the serial device-mapping into the container will get lost)
   https://github.com/mkuf/prind
  *  Never tried it myself, but would guess that **KIAUH** is probably the best option.
   it's often found in posts all over the  web: *KIAUH - Klipper Installation And Update Helper*
       https://github.com/th33xitus/kiauh 

*  *how to I configure klipper etc.?*
    *  You could start with Funkton's config files and start from there (unfortunately outdated)
       https://discord.com/channels/686160672211730511/821928891491942451/903867198441918487  
        *  **!!warning!! for X-Plus** some values are off because Funktons printer.cfg targets an X-Max - and a modified one :p
    
        Necessary changes to his printer.cfg:
        - X-Plus: for an X-Plus you have to adjust dimensions
        - Pullys: probably you have to go with with different rotation_distance (Funkton has 17T pullys, Qidi's original are 16T, I myself have 20T):
        
        ```
        [stepper_x]
        ...
        rotation_distance: 40 # 40 for 20T metallic pulleys; go with 32 for the original 16T pulleys and with 34 for 17T pullys (Sidenote: Funktons value for 17T: 33.984)
        ....
        position_endstop: 270
        ...
        position_max: 270
        ...
        
        [stepper_y]
        ...
        rotation_distance: 40 # 40 for 20T metallic pulleys; go with 32 for the original 16T pulleys and with 34 for 17T pullys (Sidenote: Funktons value for 17T: 33.984)
        ...
        position_endstop: 200
        ...
        position_max: 200
        ....
        
        [stepper_z]
        ....
        position_max: 210
        # dont put a  position_endstop here!... doing Z-Leveling will attach the correct value at the end of printer.cfg, looks like this:  #*# position_endstop = 204.394
        ....
        ```

    * or just use mine as a starting point, they are based on Funktons (see https://github.com/naxan6/QidiKlipperFlashConfig/tree/main/configs)

-  *when am I done with chapter 0?*  
   you are done with your klipper/moonraker/fluidd stack, if fluidd shows one of the following two (but no other errors)
    ```
       Printer is not ready
       The klippy host software is attempting to connect. Please
       retry in a few moments.
    ```

    which later on switches to 

    ```
       mcu 'mcu': Unable to connect
       Once the underlying issue is corrected, use the
       "FIRMWARE_RESTART" command to reset the firmware, reload the
       config, and restart the host software.
       Error configuring printer
    ```
**!!warning!!** after you can connect klipper to your printer for the first time, make sure to do nothing else, but as a first thing check your printer limits are set to save values (so your not crashing your head into the frame) and follow https://www.klipper3d.org/Config\_checks.html

# 1. Prepare the stuff necessary for flashing
*  Buy an ST-Link V2, e.g. https://de.aliexpress.com/item/32792513237.html
*  Update ST-LinkV2 Firmware (is later on forced by STM32CubeProg)
   *  Valid Link on 2021-11-17:   
      https://www.st.com/content/st\_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-programmers/stsw-link007.html#get-software
   (you have to register with email and get an download link in your inbox once)
*  Download STM32CubeProg
   * https://www.st.com/content/st\_com/en/products/development-tools/software-development-tools/stm32-software-development-tools/stm32-programmers/stm32cubeprog.html#get-software
   (you have to register with email and get an download link in your inbox once)
*  **Shut down the printer (physical power off button in the back)**
*  Unscrew and remove the lower cover so you have access to the mainboard


# 2. Connect the ST-Link to the printer
*  Update ST-LinkV2 Firmware (it's forced by STM32CubeProg)
* **Dual check you have shut down the printer (physical power off button in the back)**
* Connect the pins, they are not in the same order but very good "matchable"  
  * (see https://discord.com/channels/686160672211730511/821928891491942451/838586898023841872)
    ``` 
       ST-Link ------> Qidi Mainboard
       ==============================
       SWCLK   ------> SWC9
       SWDIO   ------> SWI7
       GND     ------> GND
       3.3V    ------> 3V
    ```
![alt text](https://media.discordapp.net/attachments/821928891491942451/838586897835622450/image0.jpg?width=1584&height=1265)
![Alt text](/images/mainboard_overview.jpg?raw=true "Mainboard")
![Alt text](/images/focusMainboardSTLinkv2.jpg?raw=true "MainboardSTLink")
![Alt text](/images/STLinkV2.jpg?raw=true "STLink")

* Settings in STM32CubeProg (see https://discord.com/channels/686160672211730511/821928891491942451/838635980272435221)  
  * Port: SWD
  * Frequency 8kHz): 4000
  * Mode: Under reset
  * Access: 0
  * Reset mode: Hardware reset
  * Shared Disabled
  * ![Alt text](/images/ST-Link_config.png?raw=true "stconfig")
* Click Button 'Connect' (top right corner) in STM32CubeProg
* case 1: you have the original Bootloader on the mainboard: it will connect straight away
  * case 2: you have klipper or something else on the mainboard: it will
   not connect straight away, you have to
    *  press and hold the reset-button on the mainboard (it's there behind
      the usb and ethernet ports, see image)
    * click and release the button 'Connect' (top right corner) in STM32CubeProg
    * release the reset-button on the mainboard
      ![Alt text](/images/resetButton1.jpg?raw=true "reset-button")
* Now it should be connected

# 3. Backup complete original firmware (including bootloader)
* execute chapter 2
* address should be at 0x08000000
* set size to 0x80000 (..which means 512K=whole flash size)
* click Read in the upper right and then next to that the down arrow and save as e.g. "InitialBackup.bin" and store multiple copies everywhere :p
* **DISCONNECT the cables from printer before switching it on again**

FYI according to https://www.st.com/en/microcontrollers-microprocessors/stm32f407-417.html flash size should be 512K (I have an "STM32F407ZE" on my X-Plus mainboards main chip)

# 4. (optional) Restore original firmware (including bootloader)

* execute chapter 2
* switch to "Download" (see https://discord.com/channels/686160672211730511/821928891491942451/838637467258060820)
* address should be at 0x08000000
* choose your previous "InitialBackup.bin"
* Click "Start Programming", wait until it's done
* Click Button 'Disconnect' (top right corner) in STM32CubeProg
* **DISCONNECT the cables from printer before switching it on again**

# 5. Compile klipper.bin for your printer and flash it

* follow https://www.klipper3d.org/Installation.html
* when doing ``make menuconfig`` follow what Funkton describes in his
   printer Config for FIRMWARE build, see excerpt of current version
   below.
    ```text
       # This is a Klipper configuration for Qidi X-Max, with
       # V4.6 motherboard. By Funkton
       ## FIRMWARE BUILD

   1. Enable extra low-level configuration options checkbox
   2. Set MCU type to STMicroelectronics STM32
   3. Choose processor model STM32F407
   4. Set No bootloader option
   5. For serial connection Uncheck USB, and leave default serial settings, USART1. For USB option check box and leave defaults
   6. For serial connection set Baud rate to 115200 (or adapter proferred). This value must match baud rate in printer.cfg file
   7. Set (PE0, !PG10) as GPIO pins to set at startup. PE0 keeps on the power module and !PG10 turns on the board LED
   8. Save changes and use make to build firmware.
   9. Connect ST-LINK to mainboard SWD header and flash firmware.bin directly into user memory at 0x8000000
    ```

* Flash the produced 'klipper.bin' with STM32CubeProg to 0x8000000
   (exactly the same steps like chapter 4, Restoring)
* sidenotes:
  - I used the option USART1 as I connected an USB-to-serial-converter Raspberry PI 3b+ to the usb of the raspberry and the serial side to the mainboard.
  - I used exactly "PE0,!PG10" (without apostrophes)

# Appendix A - as a sidenote some observations regarding the klipper firmware installation
----------------------------------------------------------------------------------------
*  looked through the Device Memory Bytes...
  * Start is at 0x08000000
  * there is a jump to FFF...FFF at 0x0800A970  maybe Bootloader itself has a size of 43.376 bytes / 42+k ???
  * there is a jump from FFF...FFF  0x08020000  maybe Bootloader ("reserved" or such ???) is 131.072 bytes / 128k ???
  * there is a jump to FFF...FFF at 0x08074000  maybe Qidi firmware is 344.064 bytes / 336k ???
           this matches the last installed firmware update which was actually 336k in size
*  compiling a firmware with 'make menuconfig'-bootloader offsets zero,
   128.5k and even straight 128k (manually added) each followed by
   conversion to an update.qd file and placing on an usb stick did not
   yield immediate success. It will install, as can be seen by
   inspecting the memory with STM32CubeProg, but does not work; maybe
   wrong entry point or such (effect: klipper can not connect)
*  script to convert a compiled klipper.bin to an update.qd, which the
   original bootloader will write to flash on startup if placed on an
   usb stick: https://github.com/Klipper3d/klipper/blob/master/scripts/update\_chitu.py.
*  note: the flash of a specific file will only happen once (you hear a
   single short deep beep) - even if bootloader itself is still present
   - the second time it will not beep and probably will do nothing.
   Switch to another update.qd version and it will work again.
