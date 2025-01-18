# Ocean-Optics-USB2000
This is a compilation of info about Ocean Optics USB2000 spectrometers and a guide to flashing them with firmware on the board level.

To start this off, the Ocean Optics USB2000 is a compact crossed Czerny-Turner spectrometer
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

***To get around this issue it is possible to directly flash the EEPROM that stores the firmware.***

To access it, follow these steps:
### Opening the lid of the USB2000:
Use a razor blade or a suitable alternative to gently peel back the corners of the lid sticker.  
I recommend doing this step carefully, both to avoid injury and to retain the spectrometer's resale value and aesthetics ;)  
If the USB2000 is missing the lid sticker, skip this step.  

Screw location shown on a unit without the lid sticker:  

![NoSticker](https://github.com/user-attachments/assets/3b1642ff-d517-4365-a463-14678c337c1f)

### If you have located all four corner screws, unscrew them.


![StickerLid](https://github.com/user-attachments/assets/3eaafccb-7831-45f8-96a5-7ccb042882c5)








