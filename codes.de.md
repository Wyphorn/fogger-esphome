[English](codes.md) | **Deutsch**

# Fogger LED-1200 – Funkcodes der Fernbedienung

Ausgelesen 2026-10-31 mit nanoCUL (CC1101, SIGNALduino 3.3.1, 433.92 MHz).

**Kein IR, sondern 433 MHz Funk (OOK).** Am Gerät ist kein IR-Empfänger.

## Protokoll
- 433.92 MHz, OOK
- Sync: ~400 µs high, ~12.9 ms low
- Bit 0: ~400 µs high, ~1200 µs low
- Bit 1: ~1200 µs high, ~400 µs low
- 32 Bit, MSB zuerst: `A7 FC <cmd> <~cmd>` (NEC-artig)
- Gehaltene Taste: ganzer Code wiederholt, ~4× pro Sekunde
- Tastencodes zählen die Plätze von oben links nach unten rechts, dezimal als Hex
  geschrieben. Platz 02 (zwischen ON und OFF) ist leer.

## Codes
| cmd | Taste | Code | Status |
|---|---|---|---|
| 01 | ON (Nebel) | 0xA7FC01FE | empfangen |
| 03 | OFF | 0xA7FC03FC | empfangen |
| 04 | Red | 0xA7FC04FB | empfangen |
| 05 | Green | 0xA7FC05FA | empfangen |
| 06 | Blue | 0xA7FC06F9 | empfangen |
| 07 | Jump | 0xA7FC07F8 | empfangen |
| 08 | Gradual change | 0xA7FC08F7 | empfangen |
| 09 | Pulse change | 0xA7FC09F6 | empfangen |
| 10 | Stroboscopic | 0xA7FC10EF | empfangen |
| 11 | Brightness ▲ | 0xA7FC11EE | empfangen |
| 12 | Brightness ▼ | 0xA7FC12ED | empfangen |
| 13 | Gelbgrün | 0xA7FC13EC | berechnet, per Draht bestätigt |
| 14 | Rosa | 0xA7FC14EB | empfangen |
| 15 | Dunkelblau | 0xA7FC15EA | empfangen |
| 16 | Hellblau | 0xA7FC16E9 | berechnet, per Draht bestätigt |
| 17 | Graulila | 0xA7FC17E8 | empfangen |
| 18 | Grün | 0xA7FC18E7 | empfangen |
| 19 | Grün (2) | 0xA7FC19E6 | empfangen |
| 20 | Dunkelblau (2) | 0xA7FC20DF | empfangen (aus MU rekonstruiert) |
| 21 | Magenta | 0xA7FC21DE | empfangen |
| 22 | Lila | 0xA7FC22DD | empfangen |
| 23 | Blauviolett | 0xA7FC23DC | Teilempfang, Prüfsumme passt |
| 24 | Light off | 0xA7FC24DB | empfangen |

## Offen
- Nebelt ON nur solange gehalten, oder bis OFF?

## Per Draht statt Funk (2026-11-17)
Funkmodul im Fogger ist eine eigene Platine am 3-pol. JST (GND / DO / 5V), 5V-Seite
per Flyback-Trafo vom Netz getrennt (GND gegen L/N gemessen: offen). Modul abgesteckt,
D1 Mini speist die Codes direkt auf DO ein (fogger.yaml). DO: Ruhe LOW, HIGH = Traeger.
3.3 V vom ESP reichen dem Controller nicht als HIGH -> NPN S8050 als Pegelwandler
(D1 -1k- Basis, Kollektor an DO + 10k nach 5V, Emitter GND, inverted: true).
D1 Mini wird aus JST 5V versorgt (+ Elko), laeuft stabil auch beim Senden.
Alle Tasten getestet.
