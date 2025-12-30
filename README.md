# Varia-AKU-bluetooth-doc
Because there’s no public documentation detailing the scale’s Bluetooth functionality, I took the initiative to reverse-engineer its UUIDs and data payloads in order to unlock the full capabilities of its Bluetooth features.

Bluetooth UUIDs
UUID Type	Value
Service UUID	FFF0
Characteristic UUID (notifications/weight data)	FFF1
Command UUID (write commands)	FFF2


Command Payloads
All payloads follow the format: [0xFA, command_byte, 0x01, 0x01, XOR_checksum]
The XOR checksum is calculated as: command_byte ^ 0x01 ^ 0x01
Command	Payload
Tare	[0xFA, 0x82, 0x01, 0x01, 0x82]
Timer Start	[0xFA, 0x88, 0x01, 0x01, 0x88]
Timer Stop	[0xFA, 0x89, 0x01, 0x01, 0x89]
Timer Reset	[0xFA, 0x8A, 0x01, 0x01, 0x89] (note: XOR uses 0x02 for last byte)


Weight Data Parsing
When receiving notifications on FFF1, the weight data is parsed as:
```typescript
// Check if byte[1] === 0x01 (weight update message)
// Sign: bit 4 of byte[3] (0 = positive, 1 = negative)
sign = (rawStatus[3] & 0x10) === 0 ? 1 : -1;

// Weight in raw units (divide by 100 for grams)
weight = sign * (((rawStatus[3] & 0x0F) << 16) + (rawStatus[4] << 8) + rawStatus[5]);
weightInGrams = weight / 100;
```
