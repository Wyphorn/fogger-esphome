**English** | [Deutsch](README.de.md)

# Fogger with ESPHome

Control a cheap fog machine (**"LED-1200 Smoke Machine"**, 1200 W, with RGB LEDs) from **Home Assistant**. A **WEMOS D1 Mini** running ESPHome is wired directly into the fogger's controller. There is no radio left in the loop.

![PCB in 3D](docs/platine_iso.png)

## How it came about

For Halloween I wanted a ghost to appear at my front door. The motion sensor triggers, the music fades in, the light gets brighter, the strobe kicks in, and then a burst of fog. The fogger only came with a cheap 24-button remote, so I had to get it into Home Assistant somehow.

**Attempt 1: infrared.** The remote looks like the typical IR remote for RGB lamps. So I hooked an IR receiver to an ESP to capture the codes. Nothing came in. The fogger has no IR receiver at all.

**Attempt 2: 433 MHz.** With a nanoCUL (CC1101, SIGNALduino) I then captured every button. It is simple OOK on 433.92 MHz with an NEC-like code `A7 FC <command> <~command>`. For Halloween the ESP sent the codes through the nanoCUL, and that worked. There were two problems, though:
- Now and then an **OFF arrived as fog ON**. ON and OFF differ in only two bits. As a workaround, the automation sent OFF every 6 s during the fog phase.
- WiFi at the front door is weak. It worked once power saving was off (`power_save_mode: none`).

**Solution: a wire instead of radio.** After Halloween I opened the fogger. The radio receiver is a separate little board on a 3-pin JST connector labelled **GND / DO / 5V**. DO carries the demodulated signal: HIGH while the remote is transmitting, LOW otherwise. I unplugged the radio module, and now the D1 Mini puts exactly this signal on DO by wire. To the fogger it looks like a remote with perfect reception:
- All 23 buttons work, including colours, strobe and brightness.
- The fogger's heater interlock stays in place.
- Nothing is transmitted over the air any more.

One stumbling block was the logic level: the controller does not see the ESP's 3.3 V as HIGH. An NPN transistor as a level shifter pulls DO up to the fogger's 5 V, and with that it worked right away.

## Hardware

| Ref | Part | Value |
|---|---|---|
| U1 | WEMOS D1 Mini (ESP8266) | |
| Q1 | NPN, e.g. S8050 | |
| R1 | Base resistor | 1 kΩ |
| R2 | Pull-up on DO | 10 kΩ |
| C1 | Electrolytic capacitor | 220–470 µF |
| J1 | JST-XH 3-pin to the fogger | |

```
JST 5V  ──┬───────────────── D1 Mini 5V ──┬── C1 +
          │                                │
         R2 10k                            │
          │                                │
JST DO  ──┴── C ┐                          │
                Q1 (NPN)                   │
D1 ── R1 1k ─── B ┤                        │
                │                          │
JST GND ──┬── E ┘                          │
          └───────────────── D1 Mini G  ───┴── C1 −
```

- The D1 Mini is powered from the fogger's 5 V, so it only runs while the fogger is on. The regulator inside the fogger gets warm, but not hot.
- The transistor inverts the signal. That is why the firmware uses `inverted: true`.
- **Don't use D3, D4 or D8.** These are boot pins of the ESP8266, and with the transistor on the base it no longer boots from them.

You'll find the schematic and the PCB (THT, also works on perfboard) in [`kicad/`](kicad/).

## Firmware

[`fogger.yaml`](fogger.yaml) is an ESPHome configuration. Before building it, copy `secrets.example.yaml` to `secrets.yaml` and fill in your details.

In Home Assistant you get:
- **Buttons** for every button on the remote: colours, effects, brightness and light off.
- **Fog burst:** ON, then OFF after an adjustable time. The sequence runs on the ESP, even if Home Assistant drops out in between.
- **Fog ON / OFF:** ON has a safety maximum duration, after which the ESP sends OFF by itself.
- **Diagnostics:** WiFi signal, uptime, reset reason.

The entity names are German (e.g. „Nebelstoß“ = fog burst, „Rot“ = red), because my automations use them. Rename them in `fogger.yaml` as you like.

All codes and the protocol are in [`codes.md`](codes.md).

### Other files
- `fogger_cul.yaml`: the Halloween version, an ESP32-C6 drives a nanoCUL over UART, which transmits.
- `fogger_c6.yaml`: the wired version for an ESP32-C6 instead of a D1 Mini, without the level shifter.
- `fogger_ir_alt.yaml`, `*_log*.txt`: the IR attempt and the capture logs.

## ⚠️ Safety

Inside the fog machine there is **mains voltage (230 V)** and a heater block at several hundred degrees. Only open it unplugged. Before connecting anything, check that the low-voltage side is isolated from mains. In my unit a flyback supply with a transformer provides the isolation, and GND to the mains plug pins measured open. Some cheap devices use a capacitive dropper instead, and there GND sits at mains potential.

Build at your own risk.

## License

[MIT](LICENSE)
