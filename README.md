# Ocean-Optics-USB2000
This is a compilation of info about Ocean Optics USB2000 spectrometers and a guide to flashing them with firmware on the board level.

### Disclaimer: I am not responsible for any damages caused by using this procedure. You can use it at your own risk.  

## Introduction
The Ocean Optics USB2000 is a compact crossed Czerny-Turner type spectrometer  
with SMA905 fiber optic input and a 10-pin IO   
(connector is Samtec IPS1-105-01-S-D-RA, avail. from Mouser).  
The IO can be used to trigger spectral captures and to trigger external devices,  
such as flashlamps, lasers etc..  
The optical design is very similar to it's predecessor, the Ocean Optics S2000,  
and has been used in later models as well. It uses the Sony ILX511 linear CCD image sensor
and can be found  
in various configurations ranging from Deep UV over vis to NIR.

Here's a sketch of the optical layout of the USB4000 spectrometer, which is pretty much identical to the USB2000's.  
Image credit: Fernando Sebastián Chiwo on ResearchGate.

![Layout_USB2K](https://github.com/user-attachments/assets/0aeec642-dd8b-4bae-9265-e9d3c2807801)

...and the real thing:  


![Layout](https://github.com/user-attachments/assets/cae66f44-db83-419b-b6c7-e5cece646956)


While it is rare to find units that are configured to measure spectra beyond 880nm,
I have modified a USB2000 to measure up to 1080nm with still acceptable sensitivity
for laser building applications.

The USB2000 spectrometer has seen use in many different applications.
Most times it is used with Ocean Optics' firmware installed and with Ocean Optics Software such as OceanView.
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

First locate the EEPROM, on which the firmware is stored. The model number is 24LC256. The EEPROM communicates via I	<sup>2</sup> C.    
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

![probing](https://github.com/user-attachments/assets/eb921588-6b9b-4bfe-9254-a1c86b63da56)


## Flashing the EEPROM

Load the firmware file `usb2000v2510.iic` onto the SD card. Now put the SD card into the Arduino SD shield.
Use the Arduino to run the following code:

```
#include <SPI.h>
#include <SD.h>
#include <Wire.h>

#define PROG_FNAME	    "USB200~1.iic"
#define PROG_VERIFY	    0

#define PIN_SDCARD_CS	    4

#define UART_BAUD	    9600

#define EEPROM_I2C_ADDR	    0x51
#define EEPROM_BASE	    0

Sd2Card card;

byte eeprom_read(int i2c_addr, unsigned int dev_addr)
{
	byte data = 0xff;

	Wire.beginTransmission(i2c_addr);
	Wire.write((int)(dev_addr >> 8));
	Wire.write((int)(dev_addr & 0xff));
	Wire.endTransmission();
	Wire.requestFrom(i2c_addr, 1);

	if (Wire.available()) 
		data = Wire.read();

	return data;
}

int eeprom_write(int i2c_addr, unsigned long dev_addr, byte data)
{
	Wire.beginTransmission(i2c_addr);
	Wire.write((int)(dev_addr >> 8));
	Wire.write((int)(dev_addr & 0xFF));
	Wire.write(data);
	Wire.endTransmission();

	return 0;
}

int prog_eeprom(const char *fname)
{
	char buf[64];
	uint8_t e_byte;
	uint8_t f_byte;
	File f;
	char c;

	Serial.println("Opening firmware file");

	f = SD.open(fname, FILE_READ);
	if (!f) {
		Serial.println("Could not open file");
		return -1;
	}

	Serial.print("Will reprogram EEPROM. Continue? [y/N]");
	while (!Serial.available()) {
		yield();
	}
	c = Serial.read();
	if (c != 'y') {
		Serial.println("Aborting");
		return -1;
	}

	Serial.println("");
	Serial.print("Programming EEPROM (");
	Serial.print(f.size());
	Serial.print(" bytes)\n");

	for(unsigned long i = 0; i < f.size(); i ++) {
		f_byte = f.read();
		eeprom_write(EEPROM_I2C_ADDR, EEPROM_BASE + i, f_byte);
		delay(10); /* usec */
#if PROG_VERIFY
		e_byte = eeprom_read(EEPROM_I2C_ADDR, EEPROM_BASE + i);
		if (f_byte != e_byte) {
			sprintf(buf, "Could not verify byte at %08x: 0x%x (should be 0x%x)", i, f_byte, e_byte);
			Serial.println(buf);
			return -1;
		}
#endif /* PROG_VERIFY */
	}
	f.close();

	return 0;
}

void setup()
{
	int rc;

	Wire.begin();

	Serial.begin(UART_BAUD);
	while (!Serial)
		; /* wait for serial monitor */

	Serial.println("Initializing SD card");

	if (!SD.begin(PIN_SDCARD_CS)) {
		Serial.println("Could not initialize SD card");
		while (1);
	}

	Serial.println("Programming EEPROM");
	rc = prog_eeprom(PROG_FNAME);
	if (rc) {
		Serial.println("Programming failed");
		while (1) ;
	}
	Serial.println("Programming complete");
}

void loop()
{
	/* stub */
}
```
After this, your USB2000 spectrometer should have been flashed with the correct firmware for it and can be used with Ocean Optics' software.  

Now you can connect the spectrometer to your PC and run the OOI USB Programmer tool by Ocean Optics.  
It should show up now and even have all the calibration data still on it (although it makes sense to at least verify the calibration after all the years - for a quick and dirty check, checking against the spectral peaks of a fluorescent lamp works. For a more refined calibration, you should use a dedicated Mercury Argon spectral lamp.)    

The calibration procedure is laid out in Appendix A on pp. 35 - 39 of the `Ocean_Optics_USB2000_Operating_Instructions_Manual.pdf`, which can be found in this repository.  

### Unlock Codes
To unlock all features of Ocean Optics' softwares, there are unlock codes.   

***!!!  These should be used carefully, as they can modify devices in non-reversible ways.  !!!***  

For the OOI USB Programmer the unlock code is "UNLOCKALL".  
For OceanView the unlock code is "oceanfactory".  

With these unlock codes, you can flash the calibration coefficients you calculated to the spectrometer.  

Good luck and have fun!


