**English** | [Deutsch](codes.de.md)

# Fogger LED-1200: remote control codes

Captured on 2026-10-31 with a nanoCUL (CC1101, SIGNALduino 3.3.1, 433.92 MHz).

**Not IR, but 433 MHz radio (OOK).** The device has no IR receiver.

## Protocol
- 433.92 MHz, OOK
- Sync: ~400 µs high, ~12.9 ms low
- Bit 0: ~400 µs high, ~1200 µs low
- Bit 1: ~1200 µs high, ~400 µs low
- 32 bits, MSB first: `A7 FC <cmd> <~cmd>` (NEC-like)
- Held button: the whole code repeats about 4× per second
- The button codes count the positions from top left to bottom right, written in decimal as hex. Position 02 (between ON and OFF) is empty.

## Codes
| cmd | Button | Code | Status |
|---|---|---|---|
| 01 | ON (fog) | 0xA7FC01FE | captured |
| 03 | OFF | 0xA7FC03FC | captured |
| 04 | Red | 0xA7FC04FB | captured |
| 05 | Green | 0xA7FC05FA | captured |
| 06 | Blue | 0xA7FC06F9 | captured |
| 07 | Jump | 0xA7FC07F8 | captured |
| 08 | Gradual change | 0xA7FC08F7 | captured |
| 09 | Pulse change | 0xA7FC09F6 | captured |
| 10 | Stroboscopic | 0xA7FC10EF | captured |
| 11 | Brightness ▲ | 0xA7FC11EE | captured |
| 12 | Brightness ▼ | 0xA7FC12ED | captured |
| 13 | Yellow-green | 0xA7FC13EC | derived, confirmed by wire |
| 14 | Pink | 0xA7FC14EB | captured |
| 15 | Dark blue | 0xA7FC15EA | captured |
| 16 | Light blue | 0xA7FC16E9 | derived, confirmed by wire |
| 17 | Grey-purple | 0xA7FC17E8 | captured |
| 18 | Green | 0xA7FC18E7 | captured |
| 19 | Green (2) | 0xA7FC19E6 | captured |
| 20 | Dark blue (2) | 0xA7FC20DF | captured (rebuilt from raw MU data) |
| 21 | Magenta | 0xA7FC21DE | captured |
| 22 | Purple | 0xA7FC22DD | captured |
| 23 | Blue-violet | 0xA7FC23DC | partially captured, checksum matches |
| 24 | Light off | 0xA7FC24DB | captured |

## Open
- Does ON only fog while held, or until OFF?

## By wire instead of radio (2026-11-17)
The radio receiver in the fogger is a separate board on a 3-pin JST (GND / DO / 5V). A flyback transformer isolates the 5V side from mains (GND to L/N measured open). I unplugged the module, and the D1 Mini now feeds the codes straight onto DO (`fogger.yaml`). DO idles LOW and is HIGH while a carrier is present.

The controller does not accept the ESP's 3.3 V as HIGH. An NPN S8050 works as a level shifter: D1 → 1k → base, collector to DO with 10k to 5V, emitter to GND, `inverted: true`. The D1 Mini runs from the JST 5V (plus an electrolytic cap) and stays stable while transmitting. All buttons tested.
