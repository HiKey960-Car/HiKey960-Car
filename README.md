# HiKey960-Car

# Parts:
(This only covers major components, if you can't figure out the rest, you should think twice about this project)
- HiKey 960 SBC: https://www.96boards.org/product/hikey960/
- Grove Seeed Sensor Mezzanine: https://www.96boards.org/product/sensors-mezzanine/
- Vantec NBA-200U: http://www.vantecusa.com/products_detail.php?p_id=155
- //Monitor: https://www.chalk-elec.com/?page_id=1280#!/HDMI-interface-LCD-touch
- //Alternative: https://www.aliexpress.com/item/CARPC-Kit-7-Inch-1024-600-LCD-High-Brightness-Dirver-Board-Capacitive-Touch-Screen-HDMI-VGA/32251750075.html
- 4-channel automotive amplifier.
- USB HUB -- powered hub preferred, especially one that can take a 12-ish volt input and has good power filtering, then you can just power it from the acc line.
- USB GPS, like a VK-162: https://www.amazon.com/Stratux-Vk-162-Remote-Mount-USB/dp/B01EROIUEW
- DMHD-1000 AMFM Radio + USB-to-RS232 cable (you can buy wires that are specific for the DMHD-1000, or make one yourself, I'll provide instructions on that)

// Monitors;
I use a 7" chalkboard from the first link. I have also tested with the alternative. Generally, I don't recommend the alternative, HOWEVER, if you already have a chinese car radio with a 7" screen, this will probably be a drop-in option to keep the existing DDIN chassis.

In general, you can use any monitor that you like, HOWEVER, the HiKey960 board is *not quite* able to reproduce the standard HDMI timings, so monitors that are extremely sensitive may not work with it.

# The Prototype

![prototype image](https://raw.githubusercontent.com/HiKey960-Car/HiKey960-Car/master/hikey960_ivi_prototype.jpg)

# Features
Well, this is a general purpose computer that runs Android that is built into your car dashboard. It does *everything* that any other car radio can do, and *everything* that every other car radio PROMISES to do (even if they can't). It is an HFP client for phone calls, it runs mapping software, plays music, opens spreadsheets, whatever you can imagine.

# Limitations
The project has progressed to a point where it can be used comfortably for daily usage. There are, however, some things that have not been quite completed to the most ideal end goals;
- The SBC does not go into deep sleep when the ACC turns off, rather it powers off entirely. Intention is to make it deep sleep, but for now, it boots in 30 seconds flat, so it isn't all that terrible.
- There are some software bugs that upstream is working on, some graphical artifacts may be visible from time to time.
- Selinux is currently set to PERMISSIVE. Work on selinux will come later once main features are all smoothed out.

# How to build Android for this project
Read this: https://source.android.com/setup/devices
Of course, changes are needed, in particular;
- Device tree from here: https://github.com/HiKey960-Car/android_device_linaro_hikey
- Kernel from here: https://github.com/HiKey960-Car/android_kernel_linaro_hikey
- Apply patches from the /patches/ path of this repository

*You don't actually need to copy in that device and kernel, rather use the upstream device and kernel, and apply patches from these ones*.

- NOW BUILD ANDROID LIKE THIS:<br>
. build/envsetup.sh<br>
lunch hikey960_car-userdebug<br>
m -j9 Launcher3<br>
m -j9

- Follow instructions in this repository to set up SENSORS MEZZANINE: https://github.com/HiKey960-Car/hikey960_extras

# Recommended add-on applications
- uG "nogapps" GmsCore
- uG "nogapps" GsfProxy
- uG "nogapps" BlankStore
- F-Droid repository

https://github.com/microg/android_packages_apps_GmsCore/wiki/Downloads<br>
https://f-droid.org/en/

# Steering wheel interface wiring
Parts needed:
- 1 grove connector. Here are 10 for $11 (2 on each wire): https://www.amazon.ca/Quality-Universal-Wearproof-connectors-compatible/dp/B01H8Q8AFK
- 2 1/4 watt 5% resistors. More on figuring out the size later.

There are 4 color coded wires on each grove connector. Black and Red are GND and 5V respectively. White and yellow are "data". Your steering wheel will have 2 or 3 wires. Bank 1, Bank 2 (maybe), and common.<br>
<br>
Hook up the common wire to BLACK on the grove. Hook up Bank 1 and Bank 2 to YELLOW and WHITE -- it doesn't matter which. Hook up one resistor from RED to YELLOW, hook up the second resistor from RED to WHITE.<br>
<br>
Plug the grove connector into position A0 on the sensors mezzanine.<br>
<br>
Your wiring is complete.<br>
<br>
*Determining the resistor size*<br>
<br>
The ADC's read voltage. You will be feeding it a voltage in the range of 0-5V. It will read 5V when the switches are all in the OFF position. When you push buttons, it will lower the voltage. You need to pick resistors that will allow the voltages to be spread out best over the range, but without getting too close to 5V where it won't read at all.<br>
<br>
First step is to measure the resistance of all the buttons on both banks. Write down the readings on the highest and the lowest.<br>
<br>
Pick a number that is right in the middle of them.<br>
<br>
My steering wheel has resistance in the range of 330-3200 ohms.<br>
Half way between them is ((3200 - 330) / 2) + 330 = 1765 ohms.<br>
<br>
That's what you're going for, as close as you can get to 1765 ohms, or whatever yours comes out to be.<br>
<br>
It does NOT have to be dead-on. Just get something reasonably close. For mine, 1.5k or 1.6k or 2.0k or 2.2k (very very common) would all be fine.<br>
<br>
*Explanation:*<br>
The formula is Vout = Vin * (R2 / (R1 + R2))<br>
We measure Vout<br>
Vin = 5.0V<br>
R2 is the resistor you are adding<br>
R1 is the button<br>
<br>
If R1 == R2, then Vout = 2.5V. Right in the middle of the range that we can read. So we pick R1 to be right in the middle in order to spread the readings out as much as possible.
