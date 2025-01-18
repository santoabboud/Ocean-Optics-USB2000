# Ocean-Optics-USB2000
This is a compilation of info about Ocean Optics USB2000 spectrometers and a guide to flashing them with firmware on the board level.

## Introduction
The Ocean Optics USB2000 is a compact crossed Czerny-Turner spectrometer
with SMA905 fiber optic input and a 10-pin IO (connector is Samtec IPS1-105-01-S-D-RA, avail. from Mouser).
The IO can be used to trigger spectral captures and to trigger external devices, such as flashlamps, lasers etc.
The optical design is very similar to the S2000 spectrometer, it's predecessor,
and has been used in later models as well. It uses the Sony ILX511 linear CCD image sensor
and can be found in various conffigurations ranging from Deep UV over vis to NIR.

Here's a sketch of the optical layout of the USB4000 spectrometer, which is pretty much identical to the USB2000's.  
Image credit: Fernando Sebastián Chiwo on ResearchGate.

![Layout_USB2K](https://github.com/user-attachments/assets/0aeec642-dd8b-4bae-9265-e9d3c2807801)

...and the real thing:  


![Layout](https://github.com/user-attachments/assets/cae66f44-db83-419b-b6c7-e5cece646956)


While it is rare to find units that are configured to measure spectra beyond 880nm,
I have modified a USB2000 to measure up to 1080nm with still acceptable sensitivity
for laser building applications.

The USB2000 spectrometer has seen use in many different applications.
Most times it is used with Ocean Optics' firmware installed and with Ocean Optics Software such a OceanView.
Thermo Fisher Scientific however used the USB2000 with custom firmware in their Nanodrop Series
Microvolume UV-vis Spectrophotometers for Biotech applications (Nanodrop 2000, uses small xenon flashlamp as broadband light source).
Unfortunately, their firmware disables use with third-party or Ocean Optics software,
and disables the possibility to directly flash Ocean Optics firmware with the OOI USB Programmer Software tool by Ocean Optics.  


***To get around this issue it is possible to...***

## Directly flash the EEPROM that stores the firmware.

You will need:  
- Arduino Uno
- Arduino (Wireless) SD shield
- 4x mini probing grabbers
- SD card
- jumper cables


To access it, follow these steps:
### Opening up the USB2000:
Use a razor blade or a suitable alternative to gently peel back the corners of the lid sticker.  
I recommend doing this step carefully, both to avoid injury and to retain the spectrometer's resale value and aesthetics ;)  
If the USB2000 is missing the lid sticker, skip this step.  

Screw location shown on a unit without the lid sticker:  

![NoSticker](https://github.com/user-attachments/assets/3b1642ff-d517-4365-a463-14678c337c1f)

### If you have located all four corner screws, unscrew them.


![StickerLid](https://github.com/user-attachments/assets/3eaafccb-7831-45f8-96a5-7ccb042882c5)  

Once the screws are removed, pull the the lid up by prying from the gap between the USB 2.0 Type B port and the lid. It should open up, revealing the circuit board.  


![20250118_182039](https://github.com/user-attachments/assets/05b8056c-d707-40d2-ad41-744653f47a93)

### Connecting to the EEPROM

First locate the EEPROM, on which the firmware is stored. The model number is 24LC256.  
The location is marked in the pitures below:  

![20250118_182135](https://github.com/user-attachments/assets/c345c3fb-77dd-4e8d-9fa6-5eef5105bf1d)

close-up view:  

![20250118_182156](https://github.com/user-attachments/assets/20154351-0ffe-4b8b-8571-499dc61cc347)

Now use a mini probing grabber to connect to the following pins:  
- VCC to a +3.3V or +5V supply
- VSS to GND
- SDA to Analog Pin 4
- SCL to Analog Pin 5

![24LC256_pinout](https://github.com/user-attachments/assets/a3878a98-8b6f-4a4a-af3b-ceceab6394df)

It should look something like this:  

![probing](https://github.com/user-attachments/assets/4b008735-c278-4008-9d42-8c1146c7e73d)

Note that in the picture GND is connected to pin 6 of the spectrometer's IO. This is an older picture; you should connect everything directly to the correct IC legs.  





