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

# Features
Well, this is a general purpose computer that runs Android that is built into your car dashboard. It does *everything* that any other car radio can do, and *everything* that every other car radio PROMISES to do (even if they can't). It is an HFP client for phone calls, it runs mapping software, plays music, opens spreadsheets, whatever you can imagine.

# Limitations
The project has progressed to a point where it can be used comfortably for daily usage. There are, however, some things that have not been quite completed to the most ideal end goals;
- HFP client is operating at 8 kHz (narrow band voice). This will be expanded to 16 kHz in the near future.
- The SBC does not go into deep sleep when the ACC turns off, rather it powers off entirely. Intention is to make it deep sleep, but for now, it boots in 30 seconds flat, so it isn't all that terrible.
- The AMFM radio works exclusively through a custom application that opens the serial hardware directly, it does not have a proper broadcast radio HAL. One will be written in the near future.
- There are some software bugs that upstream is working on, some graphical artifacts may be visible from time to time.

# How to build Android for this project
Read this: https://source.android.com/setup/devices
Of course, changes are needed, in particular;
- Device tree from here: https://github.com/HiKey960-Car/android_device_linaro_hikey
- Kernel from here: https://github.com/HiKey960-Car/android_kernel_linaro_hikey
- Edit here to read “AT+BAC=1\r”: https://android.googlesource.com/platform/system/bt/+/master/bta/hf_client/bta_hf_client_at.cc#1650

*You don't actually need to copy in that device and kernel, rather use the upstream device and kernel, and apply patches from these ones*.

- NOW BUILD ANDROID LIKE THIS:
. build/envsetup.sh
lunch 32
m -j9 Launcher3
m -j9

- Follow instructions in this repository to set up SENSORS MEZZANINE: https://github.com/HiKey960-Car/hikey960_extras

- Build and install THIS application (for AMFM radio): https://github.com/HiKey960-Car/AMFM_DMHD-1000
- Build and install THIS application (for SWI programming): https://github.com/HiKey960-Car/SWI

Both of the last 2 applications will be removed in the future. AMFM radio will be served by the AOSP Radio application, the SWI program will be moved to the device tree.
