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
Using CRC8 as a checksum should be sufficient as we do not send packets longer than 64 bytes. (or do we?)

# DEVICE_IDs

- 0x0 = generic OSD
- 0x1 = generic Vtx
- 0x2 = generic Camera
- 0x3 ... 0xF reserved for now

# generic OSD device command set:

COMMANDS  < 0x8 will have a fixed length \
COMMANDS >= 0x8 will have variable length (byte 2 = length)

## 0x0 SET REGISTER
set a given register id to a value (used e.g. for enable, video format,...)

packet length: 5
frame format: [0x80] [0x00] [REGISTER:8] [VALUE:8] [CRC:8]

Register:
- 0x00 = enable (0 = off, 1 = on)
- 0x01 = video format (0 = auto, 1 = pal, 2 = ntsc)
- [tbd]

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
- 0x01..0xFF reserved

## open points:

- this protocol is meant to be bi-directional, further improvements will follow.
- missing command for font, custom logo / graphic upload
- missing command for enter bootloader (fw upgrades)


# generic VTX command set:

This is open for discussion, should be similar to OSD



# generic Camera control command set:

This is open for discussion, should be similar to OSD


