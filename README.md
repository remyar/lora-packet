# lora-packet

A pure [node.js](http://nodejs.org/) library to decode and encode packets for LoRa/LoRaWAN<sup>TM</sup> radio communication, based on the specification from the [LoRa Alliance](https://www.lora-alliance.org/) (based on V1.0.1 Draft 3), and as used by [The Things Network](https://www.thethingsnetwork.org/)


## Why?

* LoRa packets are encrypted at the radio link level.  They could be decrypted at the radio receiver, but frequently they're transferred onwards as-is, because the radio doesn't have the crypto keys.  This library lets you handle them in your code, rather than relying on less transparent / less documented / less convenient libraries / modules / systems.
* as a debugging tool, to check and decrypt packets
* node.js is available both on the application server, and can also be available on network gateways (which are otherwise hard to write code to run on)- a single library can be used in both places / either place
* inverted use case:  you have a remote gateway, and you want to send gateway telemetry/monitoring using the same uplink channel as used by the radio, as LoRa packets - so you encode your gateway telemetry as LoRa packets & slip them into the uplink.

## Features:

* LoRa packet parsing & analysis
* MIC (Message Integrity Check) checking
* payload decryption
* decodes uplink & downlink packets, network join etc
* ability to create LoRa format packets (TODO)

## Installation

    npm install lora-packet

## Usage

### create(data)

Parse & create packet structure from buffer

### packet.getBuffers()

returns an object containing the decoded packet fields, named as per LoRa spec, e.g. *MHDR*, *MACPayload* etc

### packet.getMType()

returns the packet *MType* as a string (e.g. "Unconfirmed Data Up")

### packet.getDir()

returns the direction (*Dir*) as a string ('up' or 'down')

### packet.getFCnt()

returns the frame count (*FCnt*) as a number

### packet.getFPort()

returns the port (*FPort*) as a number

### packet.getFCtrl.ACK()

returns the flag (*ACK*) of field *FCtrl* as a boolean

### packet.getFCtrl.ADR()

returns the flag (*ADR*) of field *FCtrl* as a boolean


### verifyMIC(packet, NwkSKey)

returns a boolean; true if the MIC is correct (i.e. the value at the end of the packet data matches the calculation over the packet contents)

### calculateMIC(packet, NwkSKey)

returns the MIC, as a buffer

### decrypt(packet, AppSKey, NwkSKey)

decrypts and returns the payload (NB the correct key is chosed depending on the value of *FPort*)





## Example:

```javascript
var lora_packet = require('lora-packet');

// decode a packet
var packet = lora_packet.create(new Buffer('40F17DBE4900020001954378762B11FF0D', 'hex'));

// debug: prints out contents
// - contents depend on packet type
// - contents are named based on LoRa spec
console.log ("packet.toString()=\n"+packet);

// e.g. retrieve payload elements
console.log("packet MIC=" + packet.getBuffers().MIC.toString('hex'));
console.log("FRMPayload=" + packet.getBuffers().FRMPayload.toString('hex'));

// check MIC
var NwkSKey = new Buffer('44024241ed4ce9a68c6a8bc055233fd3', 'hex');
console.log("MIC check="+(lora_packet.verifyMIC(packet, NwkSKey) ? "OK" : "fail"));

// calculate MIC based on contents
console.log("calculated MIC=" + lora_packet.getMIC(packet, NwkSKey).toString('hex'));

// decrypt payload
var AppSKey = new Buffer('ec925802ae430ca77fd3dd73cb2cc588', 'hex');
console.log("Decrypted='"+lora_packet.decrypt(packet, AppSKey, NwkSKey).toString()+"'");
```

## Notes:

#### Endianness

* LoRa sends data over the wire in little-endian format  (see spec #1.2 "The  octet  order  for  all  multi-­octet  fields  is  little  endian"
* lora-packet attempts to hide this from you, so e.g. DevAddr & FCnt are presented in big-endian format.  For example, DevAddr=49be7df1 is sent over the wire as 0xf1, 0x7d, 0xbe, 0x49.


### Can I help?

* I've done some testing, but of course I can only test using the packets that I can generate & receive with the radios I've got, and packets I've generated myself.  If you find a packet that `lora-packet` fails to parse, or incorrectly decodes any packet, please let me know!


s