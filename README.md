# MultiWii Configurator (MultiWiiConf)

 A fork of [version 2.4 of the MultiWii Configurator](https://github.com/xdu-aero-association/MultiWii_2_4). Updated for Processing 3, to allow testing with the NexgenMSP (MultiWii Serial Protocol) Arduino library.

 The MultiWii Configurator was designed for configuring, simulating and calibrating Radio Control (RC) vehicles which communicate using the MultiWii Serial Protocol (MSP).

 ## MultiWii Serial Protocol (MSP)

The MultiWii Serial Protocol was originally created for use with the MultiWii Configurator (a Processing application like the Nexgen Configurator) to control a multirotor RC model. The first draft of the protocol was defined in 2012 and was was initially developed to support Nintendo Wii console gyroscopes and accelerometers (hence the name).

It is now used by a number of open source flight controllers like ArduPilot, the BetaFlight/CleanFlight family, HackFlight and iNav.

ArduPilot supports the MSP protocol for telemetry and sensors via any of its serial ports. This allows ArduPilot to send its telemetry data to MSP compatible devices, such as DJI goggles, for On Screen Display (OSD).

The original design objectives for the MSP were:
- Light weight.
- Generic (i.e., used transparently by a GUI, OSD, telemetry or home made configuration tool).
- Bit wire efficient -  only sends requested data which is transmitted in a binary format.
- Secure - data is sent with a checksum, preventing corrupted configuration being injected.
- Header sensitive - as it is designed with a specific header, it can be mixed with other frames, like GPS. It should be possible to connect either a GUI or a GPS to the same serial port without changing configurations.
- Backwards compatible. It should be possible to add new commands without breaking previous versions of the protocol.
- Variable data length to future proof the protocol (e.g., to add a new PID controller).

There are three types of MSP message that can be sent:
- Command - a message sent to the flight controller which has some information to be sent.
- Request - a message sent to the flight controller asking for some information to be returned.
- Response - a message sent by the flight controller with information responding to a request.

MSP messages have a specific structure. They have a header, size, type, data and checksum, in that order.

## Header

The MSP header is 3 bytes in length and has two start bytes `$M` followed by the message direction (`<` or `>`). 

- `<` - From the flight controller (FC →); and
- `>` - To the flight controller (→ FC).

## Size

The fourth byte is the length (in bytes) of the data payload. For example, if the data section had three `INT 16` variables then the size byte would be 6. This field can be zero as in the case of a data request from the Configurator.

## Type

The type byte is similar to our command/request byte. It defines the command to the drone, request for information or the type of response. A list of the original MSP command types is provided in the reference folder of the NexgenMSP library. 

The incoming message flow is composed of commands and requests while the outgoing flow contains responses. 

The message ID value of each command uses the convention:

- Value `1xx` identify requests; while 
- Value `2xx` identify commands.

By agreement, the message ID range from 50–99 won't be assigned in future versions of MSP and can therefore be used for any custom multiwii fork without fear of MSP ID conflict. 

## Payload

The data payload depends on the type request. An example is the data request MSP_IDENT. This returns three `uint8_t` and one `uint32_t` bits of data. A full list of the returned data types is provided in the reference folder of the NexgenMSP library. Multi-byte data (e.g., `uint16_t`) is transmitted LSB first.

## Checksum

The checksum is computed as the `XOR` of the size, type and payload bytes. The checksum of a request (i.e, a message with no payload) equals the type.


