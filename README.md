# Dual USB Controller adapter
This project is an interface board to connect two USB controllers to the DB9 controller ports of an Atari ST/Amiga system.

This is a sequel to the "USB Controller adapter" project : it provides two USB ports and requires less components.

Working so far :
- two USB ports for two USB HID devices like mouse, keyboard or joystick/gamepad
- two DB9 outputs to the controller port of the Atari ST/Amiga
- DB9 input to daisy chain an Atari controller
- self-power or external power
- gamepad compatibility
  - Tested with wired Xbox 360 gamepad, PS3 gamepad and HID joystick/gamepad.
  - CD32 buttons
  - Autofire on right trigger
  - 5 switchable modes on Xbox 360 and PS3 gamepad
- keyboard support
  - single USB keyboard to replace two gamepads, but you can also use two keyboards
- mouse support
  - wheel mouse with the provided newmouse driver and suitable applications

Limitations :
- other HID gamepads/joysticks should also work but button placement is device dependant.
- composite devices that report multiple interfaces (eg mouse and keyboard) work only with the first interface/device.

# Disclaimer
This is a hobbyist project, it comes with no warranty and no support. Also remember that the Amiga machines are about 30 years old and may fail because of such hardware expansions.

I publish my work under the CC-BY-NC-SA license. The content of this repository is the result of weeks of work, requiring investment in tools, parts, prototypes, and risky tests on his own equipment. For this reason the author does not want third parties to sell products and keep all the profit, usually without even offering support to their customers. If you see people selling this without the consent of the author, don't buy from them.

If you find it useful and want to reward me : I am always looking for Amiga/Amstrad CPC/Commodore(...) hardware to repair and hack, please contact me.

# Power, ESD and other Warning
The adapter is self-powered by default : the host must provide 5V through its port(s) and enough power for the adapter and the USB controllers. Even if they are given for 50ma, a single port of the A500, A600, A1200, and 520STF I tested is OK for the max 130mA required by my mouse, keyboard or Xbox 360 controller. However you use it at your own risk on your system.

You can power it externally : to disable self power, you must cut the trace on the PCB between the 2 pads of the jumper then solder the "Power" pin header. Pin 1 is GND and pin 2 is 5V. When you want to self-power again, you then can install a jumper between pins 2&3 in the power pin header.

If you plan to use this adapter on other host systems, you must pay attention to the DB9 pinout that might need special wiring.

The adapter's software itself supports hot-plug both on DB9 and USB port, however there is no hardware electrical protection. The manuals of the host systems warned to always turn it off before un/plugging any device. Again, you do it at your own risk.

# BOM
- 1x STM32F205RBT6
- 1x 3.3v LDO, either SPX3819M5-L-3-3 or XC6206P332MR
- 2x 1uF (or more) 0805 capacitors
- 2x 2.2uF 0805 capacitors
- 5x 100nF 0805 capacitors
- 4x 22ohm 0805 resistors
- 2x 470ohm 0805 resistors **
- 1x 4.7K 0805 resistor
- 2x 1M 0805 resistors **
- 2x 10K 0603x4 resistors array
- 2x USB A female socket, type G54
- 2x DB9 female sockets, 1x DB9 male socket ***
- 1x Self-locking 5.8mm switch ****
- D1 led and R3, D2 and R4 are optional
- The SW2 button, Y1 crystal and C1, C2 capacitors are not used

Power and programming headers are optional : just maintain the connection during the 5 seconds of the programming operation. It will also be easier to put the device in an enclosure without it.

** Required only if CD32 buttons emulation is needed.
*** I recommend using two "Genesis gamepad extension cords" extension cables cut in half to be compatible with A600/Atari ST and to avoid metallic shielding that can make shorts when connecting to a live system. Otherwise you can solder standard plugs with the PCB board sandwiched between the pins.
**** You can hard-wire the switch pins if you only use an Atari of leave it open for an Amiga. Install a switch if you need both.

# Making it
Components are SMD, and have thin pin pitch. You need to know what you are doing. Check for shorts at least between 5V, 3.3V, and GND traces before applying power !

Since 06-2023 the firmware is made of two parts to ease future application update by users :
- A bootloader, that must be flashed at address $08000000 in the MCU using dedicated tools such as stlink with the open-source tools (https://github.com/stlink-org/stlink) or STM32CubeProgrammer, or TTL serial adapter with stm32flash.
- The application, that is flashed by the bootloader and can be updated later on : place the .upd file at the root of a FAT USB mass storage device, plug the mass storage in port A, and power up the adapter for at least 10s or until the LED of the USB key stops flashing.

The bootloader and application firmware are compiled using STM32CubeIDE. Amiga software is compiled with VBCC and AmigaOS NDK.

# Using it
To keep things simple to use, each DB9 port has a preferred USB input : 1.Mouse with USB A and 2.Game with USB B.

This means you should connect 1.Mouse to the mouse port of the host and 2.Game to the game port, then the USB mouse in port A and gamepad in port B. The layouts are swapped when plugged in the other USB port.

Here is more information about the USB devices you can connect.

## USB Mouse
Supports USB1 or USB2 mouse, and emulates a 2 buttons mouse.
Mouse movements are sent to the preferred DB9 output only. A USB mouse only affects its preferred DB9 port.

Wheel and wheel button are supported on the 1.Mouse/USB A port when using the provided "driver" for AmigaOS and "newmouse" compatible applications.

## USB Keyboard
Supports USB keyboard in boot mode, and emulates at the same time two controllers and 2-buttons mouse on both ports.
- Gamepad on preferred port : AQOP or AQRT + Space or left Alt + Enter or Y
- Gamepad on other port : Keypad 7456 + 0 or 1 + Enter or '+'
- Mouse on preferred port : Arrows + Space or left Alt + Enter or Y
- Mouse on other port : Arrows + keypad 0 or 1 + Enter or '+'

## Xbox 360 and PS3 gamepad
Tested with wired USB Xbox 360 original and clones, and wired USB PS3 clones. Emulates a CD32 pad and a mouse depending on the active mode.

The round button and its leds are used as a mode selector : press the round button to change mode.
5 modes are implemented by default, a few others are available in the source code.

1. mouse + pad : default when plugged in USB A
  * Preferred port = Mouse : left stick, CD32 pad buttons, autofire on right trigger, slow on left hat
  * Other port = Game : right stick, left button is right stick hat and right button is back, autofire on left trigger
2. pad + mouse : default when plugged in USB B
  * Preferred port = Game : left stick, CD32 pad buttons, autofire on right trigger
  * Other port = Mouse : right stick, left button is right stick hat and right button is back, autofire on left trigger
3. mouse + pad : for mouse operation when plugged in USB B 
  * Other port = Mouse : left stick, CD32 pad buttons, autofire on right trigger, slow on left hat
  * Preferred port = Game : right stick, left button is right stick hat and right button is back, autofire on left trigger
4. platform : for racing and platform titles when you prefer to use a button for jump or accelerate.
  * Preferred port = Game : left/right on left stick and cross, A/B for UP/DOWN, X/Y for FIRE1/2, autofire on right trigger
  * Other port = Game : right stick, left button is right stick hat and right button is back, autofire on left trigger
5. 2 players on one controller
  * Preferred port = Game : left part of the controller, 90° counter-clockwise, autofire on left trigger
  * Other port = Game : right part of the controller, 90° clockwise, autofire on right trigger

## Other gamepad
May work with standard HID sticks and gamepads : this emulates a CD32 pad on left stick to the preferred DB9 port, and mouse on right stick to the other DB9 port.
Since they are not normalized inside the USB documentation, right stick and buttons position will vary between controllers. Some even have strange things in their report descriptor which may also break their compatibility. Others need a pressing an analog/digital button on the gamepad itself.

