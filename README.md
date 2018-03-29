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
[UBLOX_GPS_HAL=TRUE] [DMHD1000=TRUE] [HDMI_RES="{mode}"] m -j9 Launcher3<br>
[UBLOX_GPS_HAL=TRUE] [DMHD1000=TRUE] [HDMI_RES="{mode}"] m -j9<br>
*note: set variable UBLOX_GPS_HAL=TRUE to enable Ublox-specific GPS hal.*<br>
*note: {mode} should be something along the lines of 1280x600@60 or 1024x600@70, etc.*<br>
*note: set variable DMHD1000=TRUE to enable AMFM Radio HAL for DMHD-1000

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

# Dash Camera
Parts needed:<br>
*One or more USB video cameras that conform to USB Video Class.*<br><br>
My recommendation would be WIDE ANGLE lenses, like 170-180 degree "fisheye". Preferably the primary (front?) camera should have native h264 encoding capability. Additional cameras should support MJPEG.<br>
My personal preference is for OPEN CIRCUIT BOARD cameras, which can be screwed in to the front facing side of the rear-view mirror to avoid interfering with your view.<br>
Here is an example of a highly suitable camera; https://www.amazon.ca/ELP-Webcams-Surveillance-Customized-fisheye/dp/B0196BPZ1C<br>
<br>
If the camera has a built-in microphone, it can also support *audio capture*.<br>
<br>
Software:<br>
*ffmpeg*<br><br>
You will need to install a suitable STATIC or ANDROID build of ffmpeg, build it yourself if you feel like it, or use the officially supported static linux build available here;<br>
https://www.ffmpeg.org/download.html#build-linux<br>
Note: You will need the ARM64 build.<br>
Extract the archive, push the ffmpeg binary to /system/bin/, and chmod 755 it.<br>
<br>
Camera setup:<br>
Plug in your Camera(s) to your USB hub.<br>
Note: It is important that your USB topology must remain static once the software setup is complete. The software will identify specific cameras based on the physical path of the devices.<br>
<br>
Once the cameras are plugged in (actually plug them in one at a time so that you know which one is which), look at the symlink targets in the path /sys/class/video4linux/<br>
Each entry will look something like this;<br>
video1 -> ../../devices/platform/soc/ff200000.hisi_usb/ff100000.dwc3/xhci-hcd.0.auto/usb1/1-1/1-1.2/1-1.2.1/1-1.2.1.4/1-1.2.1.4:1.0/video4linux/video1<br>
You need to select the final sub-path before /video4linux. In this case, you are looking for "/1-1.2.1.4:1.0/". You need that specific string, including the leading and trailing "/".<br>
<br>
The camera daemon receives its configuration via system properties. It recognizes up to 4 cameras, and 1 sound card.<br>
The properties are as follows;<br>
persist.dashcam.enabled<br>
persist.dashcam.X.path<br>
persist.dashcam.X.params<br>
persist.dashcam.X.sub<br>
persist.dashcam.audio.path<br>
Where X is one of front, rear, left, right. The only two that are mandatory are persist.dashcam.enabled and persist.dashcam.front.path<br>
<br>
To enable dashcam, run;<br>
setprop persist.dashcam.enabled 1<br>
<br>
To enable front camera, run;<br>
setprop persist.dashcam.front.path "/1-1.2.1.4:1.0/"<br>
(update value to match YOUR topology)<br>
<br>
If you need to modify the parameters of the front camera, run;<br>
setprop persist.dashcam.front.params "-input_format h264 -video_size 1280x720"<br>
<br>
Note that each camera could present more than one device. The suggested camera above for example, presents TWO. The first for RAW and MJPEG, the second for H264. If you need to select the second, use;<br>
setprop persist.dashcam.front.sub 1<br>
(the default/unset value is "0" for the first camera)<br>
<br>
Obtaining the audio path works exactly the same way, except you need to look in /sys/class/sound/, and you are looking only for links starting with "dsp".<br>
<br>
Once all parameters are set, reboot and it will begin recording.<br>

# RTC (Real Time Clock)
The HiKey960 does have a built-in RTC, however, it unfortunately does not have a battery backup, and does not even remember its time after being rebooted (even if power is not disturbed). As a result, if the device will be operating standalone (i.e., without any connection to a network with access to predefined NTP servers), then it is necessary to add on a battery backed RTC. It can also be helpful during startup for a quicker GPS lock if you have it even with a fairly reliable network connection, or if there is a chance you will be driving somewhere that there may not be any cellular coverage.<br>
<br>
There is provision added for two RTC units, both of which are available with "grove" connectors (i.e., they are modules that plug in to the sensors mezzanine). These are DS1307 (available as "Seeed RTC"), and PCM85063 (available as "Seeed High Precision RTC"). I generally recommend the latter, not because it is high precision, but because it doesn't need to be *modified* in order to work. The DS1307 does not have enough pull to bring the data line below the "low" threshold unless you remove R1 and R2 from the board.<br>
<br>
Plug the modified DS1307 Seeed RTC into either of the I2S0 plugs (5V operating at 100 kHz), or plug the PCM85063 into either of the I2S1 plugs (3.3V operating at 400 kHz). Once booted and with a valid time, perform an initial RTC configuring by running the command "hwclock -w -u -f /dev/rtc1".
