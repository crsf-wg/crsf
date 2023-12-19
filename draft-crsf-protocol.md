---
title: CRSF Protocol Specification
---

## Introduction

The CRSF protocol is used as a communication protocol within the
remote controlled vehicle ecosystem to transport informations to and
from the control handsets, radio communication modules, receivers,
flight controllers, sensors or any other component involved in such
systems.

### Terminology

[RFC 2119]: https://www.rfc-editor.org/rfc/rfc2119

In this document, the key words "MUST", "MUST NOT", "REQUIRED",
"SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY",
and "OPTIONAL" are to be interpreted as described in BCP 14,
[RFC 2119] and indicate requirement levels for compliant CRSF
implementations.

## CRSF Use Scenarios

The following sections describe some aspects of the use of CRSF. The
examples were chosen to illustrate the most common applications of
CRSF, not to limit what CRSF may be used for. The communication is
carried out over half- or full-duplex serial links, most often over
UART.

### Interface to a RF Module

At the handset side, CRSF can be used to communicate between the
handset and the RF module(s) to exchange control link information,
model control data, autopilot control data, status data (the latter
includes also model telemetry), as well as binary data (e.g. firmware
updates).

### Between a RF Receiver and an Autopilot

At the model side, CRSF can be used to communicate between the RF
receiver(s) and an Autopilot to exchange control link informatation,
model control data, autopilot control data, status data, as well as
binary data (e.g. firmware updates).

### Between peripherals and an Autopilot

At the model side, CRSF can be used to communicate between various
peripherals (sensors, actors) and an Autopilot to exchange control
information, status data and binary data (e.g. firmware updates).

### Between peripherals and control handset

At the handset side, CRSF can be used to communcicate between the
handset and various peripherals (sensors, actors) to exchange control
information, status data and binary data (e.g. firmware updates).

## Definitions

Throughout this document the term "Autopilot" is used to denote the
logic engine used to stabilize and control the movement of an R/C
craft, irrespective of the craft type (in aerial models, the Autopilot
is more often called a Flight Controller. This document avoids the
usage of a term Flight Controller to explicitly keep the craft type
generic).


## Protocol Definition

The CRSF protocol is based on serial communication (either full or half
duplex). Each use scenario defines some properties as to how the serial
communication is controlled.

### Physical Layer

The CRSF protocol uses serial communication with either of the following
physical layers:

- Full duplex idle-high UART using +3.3V signal levels,

- Half duplex idle-low UART using +3.3V signal levels.

Where possible, full duplex idle-high SHOULD be used, while half-duplex
idle-low is only considered in the communication between the handset
and transmitter modules when there is no other possibility.

#### Bit/Byte Order

Data are always transfered with the least significant bit (LSb) first. 

#### Default Bitrate

Without any prior knowledge about the capabilities of other components,
a bit rate of 416600 bit/sec shall be used.

### Frame Format

The CRSF protocol uses a generic frame format for all frames exchanged.

```
  0  1  2  3  4  5  6  7 
+--+--+--+--+--+--+--+--+
|      DESTINATION      |
+--+--+--+--+--+--+--+--+
|        LENGTH         |
+--+--+--+--+--+--+--+--+
|         TYPE          |
+--+--+--+--+--+--+--+--|
/        PAYLOAD        /
/                       /
+--+--+--+--+--+--+--+--+
|          CRC          |
+--+--+--+--+--+--+--+--|
```

Where:

- Destination (8 bits): component addressed by the frame (ex: handset,
  RF module, flight controller, etc).

- Length (8 bits): frame length in bytes not including the first 2
  fields (destination and length). Assuming an empty payload, the
  minimum value is 2 bytes.

- Type (8 bits): frame type.

- CRC (8 bits): CRC-8 (poly 0xD5) computed over type field and payload
  (`length - 1` bytes)

### Addressing

The destination field used in frames can either address a specific
component or any component by using the broacast address. Specific
addresses depend on the role of the component and do not consider
multiple instances of the same component participating on the serial
bus.

The defined addresses are as follows:

| Address | Component          |
|---------|--------------------|
| `0x00`  | Broadcast          |
| `0xEA`  | Handset            |
| `0xEC`  | Receiver           |
| `0xEE`  | Transmitter module |
| `0xC8`  | Autopilot          |

>Please note that `0xC8` seems to be used as well to transmit frames
>from handset to a transmitter module.

### Frame Types

| Frame Type | Description           |
|------------|-----------------------|
| `0x02`     | GPS data              |
| `0x07`     | Vario data            |
| `0x08`     | Battery data          |
| `0x09`     | Barometer data        |
| `0x14`     | Link quality data     |
| `0x16`     | Channel data          |
| `0x1C`     | Receiver telemetry    |
| `0x1D`     | Transmitter telemetry |
| `0x1E`     | Altitude data         |
| `0x21`     | Flight mode data      |
| `0x28`     | Ping devices frame    |
| `0x29`     | Device info           |
| `0x2A`     | Request settings      |
| `0x2B`     | Parameter settings    |
| `0x2C`     | Parameter read        |
| `0x2D`     | Parameter write       | 
| `0x32`     | Command               |
| `0x3A`     | Handset synch         |
| `0xC8`     | UART sync             |


### Extended Frame Format
