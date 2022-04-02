# Stupid Simple PixMap format. Image storage without the complexity.

## Features
 - Maximum 18 quintillion pixels each way (64 bit unsigned limit)
   * 340 undecillion pixels (340 billion billion billion billion - yes thats 4 billions)
   * 1.361 x10^15 yottabytes, which is 100x the total amount of information in the biosphere
   * TL;DR, we can be stupid simple yet not need any boundaries
 
 - Super easy to parse and emit
   * No unnecessary variations (cough cough .BMP)
   * No C syntax weirdness (cough cough XPM)
   * Simple, short binary header followed by a payload
 
 - Supports ARGB with one byte per channel per pixel


## Spec

All numbers are big endian.
Offsets and sizes are in bytes in hex.

The header should follow the format defined as below:
| Offset | Size | Example                   | Description |
|-------:|-----:|---------------------------|--|
|   `00` |  `4` | `73 73 70 6D`             | Magic bytes. Must always be `sspm` in ASCII. |
|   `04` |  `8` | `00 00 00 00 00 00 02 00` | Width. Example is `512px`. |
|   `0C` |  `8` | `00 00 00 00 00 00 01 00` | Height. Example is `256px`. |

Then the rest of the file should be a stream of binary pixels. each as so:
| Size | Example | Description |
|-----:|---------|-------------|
|  `1` | `7F`    | Alpha.      |
|  `1` | `9D`    | Red.        |
|  `1` | `65`    | Green.      |
|  `1` | `FF`    | Blue.       |

For those interested, the example colour here is Monokai purple at 50% opacity.

And that's it. The entire SSPM format. Nothing more to it really.

## Bonus compression technique
**THIS IS NOT SPEC, NOT REQUIRED FOR ENCODERS/DECODERS, BUT WORTH IMPLEMENTING IF YOU CAN DO EASILY**

 1. Omit the `sspm` magic bytes from the header.
 2. Compress via Zstandard.
 3. Prepend `sspz` magic bytes.

 1. Decoder detects and removes `sspz`.
 2. Decompress via Zstandard.
 3. Decode as usual, except not expecting `sspm` magic bytes.

## Example

<img id="ex-img" src="https://github.com/yellowsink/sspm-format/raw/master/example.png" alt="example image (in PNG)" width="50" height="50" />
(sorry for ugly interpolation, nothing I can do about it in GFM)

```
// header
73 73 70 6D
00 00 00 00 00 00 00 05
00 00 00 00 00 00 00 05
// payload
7F 6C B3 D3  7F 6C B3 D3  7F 6C B3 D3  7F 6C B3 D3  7F A5 DC F0
7F 6C B3 D3  7F 6C B3 D3  7F 6C B3 D3  7F A5 DC F0  FF C2 F1 FF
7F 6C B3 D3  7F 6C B3 D3  7F A5 DC F0  FF C2 F1 FF  FF C2 F1 FF
7F 6C B3 D3  7F A5 DC F0  FF C2 F1 FF  FF C2 F1 FF  FF C2 F1 FF
7F A5 DC F0  FF C2 F1 FF  FF C2 F1 FF  FF C2 F1 FF  FF C2 F1 FF
```
