# openTCO : an open vTx, Cam and Osd control protocol

This is a work in progress in order to assemble a lightweight and \
open serial protocol for generic Video TX, Camera and OSD control.

I am going to implement this on my upcoming 
[tinyOSD](https://www.youtube.com/watch?v=USWiuCVAzIQ) project.

**This is a draft and open for discussion.**
Please fork this repo and submit a pull request for discussion.

# Frame format

[HEADER:8 0x80] [DEVICE_ID:4 COMMAND:4]  ..data.. [CRC:8]

The total frame length is variable in order to efficiently allow short commands.
This will use CRC8/DVB-S2 as a checksum (see crc8.h for a fast example implementation).
Using CRC8 as a checksum should be sufficient as we do not send packets longer than 64 bytes. (or do we?)

# Connection

This protocol will use a full duplex serial port @115200 8N1.\
For maximum flexibility multiple openTCO devices can share the 
same serial port or run on individual serial ports.\


# DEVICE_IDs

- 0x0 = generic OSD
- 0x1 = generic Vtx
- 0x2 = generic Camera
- 0x3 ... 0x7 reserved for now
- 0x8 = 0x8 | 0x0 = generic OSD response
- 0x9 = 0x8 | 0x1 = generic VTX response
- 0xA = 0x8 | 0x2 = generic CAM response

# mandatory command set for ALL openTCO devicetypes

In order to allow automatic detection all openTCO devices will have to implement\
the register access command (0x0):

## 0x0 REGISTER ACCESS

read or write a given register id to a 16bit value (used e.g. for enable, video format,...)

packet length: 6
frame format: [0x80] [0x00] [R/W:1 REGISTER:7] [VALUE_LO:8] [VALUE_HI:8] [CRC:8]

R/W:
- 1 bit read/write flag
 - 0 = WRITE
 - 1 = READ
  - this will require a device reply (within 100ms timeout)
  - response: [0x80] [<deviceid_response>:4 0:4] [0:1 REGISTER:7] [VALUE_LO:8] [VALUE_HI:8] [CRC:8]

Mandatory registers:
- 0x00 = enable status / capabilities  [see individual device types for details]

# generic OSD device command set:

COMMANDS  < 0x8 will have a fixed length \
COMMANDS >= 0x8 will have variable length (byte 2 = length)

## 0x0 REGISTER ACCESS

see above, all devices have to support this

Register 0x00-0x0F:
- 0x00 = enable status / capabilities:  
 - b0 = enable osd
 - b1 = enable stickoverlay
 - b2 = enable spectrum
 - b3 = enable crosshair
 - b4..b15 reserved
 - bf will try to enable all capabilities by a write. when reading reg 0x00 devices should only return those capabilities that are enabled and supported
- 0x01 = video format 
 - 0 = auto, 1 = pal, 2 = ntsc
- 0x02 = overlay brightness black 
 - 0..100 in percent -> 0 = darkest (= black), 100 = brightest (= some kind of grey)
- 0x03 = overlay brightness white
 - 0..100 in percent -> 0 = darkest (= grey), 100 = brightest (= white)
- 0x04..0x0F reserved

## 0x1 FILL SCREEN REGION
Fill a given region with the given value.\
Region starts at [x,y], will size given by width and height.\
Width and Height exceeding the physical screen region has to be clipped at device level!\

packet length: 8
frame format: [0x80] [0x01] [X:8] [Y:8] [WIDTH:8] [HEIGHT:8] [VALUE:8] [CRC:8]

## 0x2 WRITE 

Write a single character to a given value
packet length: 6
frame format: [0x80] [0x02] [X:8] [Y:8] [VALUE:8] [CRC:8]


## 0x3 .. 0x7 RESERVED

## 0x8 WRITE BUFFER (horizontal auto increment)
Write n characters to the given position with horizontal auto increment.

packet length: 4 + DATA{2}
frame format: [0x80] [0x08] [LEN:8] [X:8] [Y:8] [LEN-2*DATA:8] [CRC:8]
with LEN = 0..60 (as the serial buffer is 64 bytes)

The x position will auto increment for every byte in the buffer,
y position will increment when the end of line is reached and x is set to zero.

## 0x9 WRITE BUFFER (vertical auto increment)
Write n characters to the given position with vertical auto increment.

packet length: 4 + DATA{2}
frame format: [0x80] [0x09] [LEN:8] [X:8] [Y:8] [LEN-2*DATA:8] [CRC:8]
with LEN = 1..60 (as the serial buffer is 64 bytes)

The y position will auto increment for every byte in the buffer,
x position will increment when the end of line is reached and y is set to zero.

## 0xA..0xE reserved

## 0xF SPECIAL COMMAND
special commands for gui effects (stick positions, render a spectrum, ...)

packet length: 4 + DATA{2}
frame format: [0x80] [0x0F] [LEN:8] [SPECIAL_CMD:8] ... [CRC:8]

SPECIAL_COMMAND:
- 0x00 : sticks and state, LEN=5, DATA = [A:8] [E:8] [T:8] [R:8] [FC_STATE:8]
- 0x01 : gyro fft bin data, LEN=17, DATA = [AXIS:8] [16 x GYRO FFT DATA:8]
- 0x02..0xFF reserved

## open points:

- this protocol is meant to be bi-directional, further improvements will follow.
- missing command for font, custom logo / graphic upload
- missing command for enter bootloader (fw upgrades)


# generic VTX command set:

This is open for discussion, should be similar to OSD



# generic Camera control command set:

This is open for discussion, should be similar to OSD


