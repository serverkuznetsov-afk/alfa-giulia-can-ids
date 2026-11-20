# Alfa Romeo Giulia CAN IDs

Decoded CAN bus signals for the **Alfa Romeo Giulia (2017, 2.0T, Q4 AWD, ZF 8HP automatic)**, read from the
**OBD-II port** (pins 6/14, 500 kbit/s), with the recordings I found them in.

I worked all of this out by recording the bus while doing one thing at a time (shift, brake, rev, set the
HVAC...) and writing a marker, then matching bytes to the markers and to values I asked the engine computer for
over OBD-II while driving. Each signal says how sure I am:

* **CONFIRMED**: matches the markers or OBD values across several sessions
* **LIKELY**: fits well, not fully proven
* **GUESS**: a lead, needs checking

Other Giulia / Stelvio years and engines probably share most of this (same Giorgio platform), but I've only
tested it on my car above. **Corrections and new finds are welcome.** Open an issue or a pull request.

| File | What |
|---|---|
| [`giulia_can_ids.txt`](giulia_can_ids.txt) | Plain-text list of every ID, signal, formula and note (Notepad friendly) |
| [`giulia.json`](giulia.json) | The same as a machine-readable decoder |
| [`captures/`](captures/) | 29 recordings (4.3 million frames) with markers, VIN removed. See [`captures/README.md`](captures/README.md) |

## How to read a signal

* Bytes are numbered 0-7. Signals are **big-endian**: take `bits` bits from the integer that starts at `byte`,
  after shifting right by `shift`. Then `value = raw x scale + offset`.
  Example: speed is `byte 0, 13 bits, shift 5` = `((b0 & 0x03) << 11 | b1 << 3 | b2 >> 5)`.
* Most 10-20 ms messages are protected: **byte 6 low nibble = rolling counter (0-15)** and
  **byte 7 = CRC-8 SAE J1850** (poly 0x1D, init 0xFF, xor-out 0xFF) over bytes 0-6.
  Protected IDs: 0DE (counter b4, CRC b5), 0EE, 0F0, 0F4, 0FA (CRC only), 0FB, 0FC, 0FE, 0FF, 100, 101, 103, 104,
  107, 116, 11C, 15A, 192, 1EF, 1F4, 1F7, 1FB, 2FA, 384, 418, 419, 5A7, 5A8, 5E0.

## Signals

| ID | Signal | Where | Formula | Unit | Confidence |
|---|---|---|---|---|---|
| 0x0FC | Engine RPM | b0-b1, 14 bits >> 2 | raw | rpm | CONFIRMED |
| 0x101 | Vehicle speed (true) | b0-b2, 13 bits >> 5 | raw / 16 = km/h (x 0.0388357 = mph) | mph | CONFIRMED |
| 0x101 | Speedometer as shown on the dash | same field | mph x 1.025 + 0.3 | mph | LIKELY |
| 0x2EF | Engaged gear | b0 high nibble | see gear table | | CONFIRMED |
| 0x2EF | Target gear (shift being prepared) | b0 low nibble | same table, F = none | | GUESS |
| 0x2EF | **Boost (gauge)** | b3 + b4 bit 7, 9 bits >> 7 | raw x 15 mbar (x 0.2176 = psi) | psi | CONFIRMED |
| 0x412 | Engine load | b0 | raw x 100/255 | % | CONFIRMED |
| 0x412 | **Accelerator pedal** | b3 | raw x 100/255 (~20 released, ~84 floored, ~90 kickdown) | % | CONFIRMED |
| 0x0DE | **Steering angle** | b0-b1, 16 bits | raw x 0.1 - 720 (left = +) | deg | LIKELY |
| 0x0DE | Steering speed? | b2-b3, 12 bits | raw - 2000 | raw | GUESS |
| 0x1F7 | Gearbox input (turbine) speed | b4-b5, 16 bits | raw x 0.5 | rpm | LIKELY |
| 0x1F7 | Transmission fluid temp? | b2-b3, 6 bits >> 3 | raw | °C | GUESS |
| 0x2ED | Coolant temp | b0 | raw - 40 | °C | CONFIRMED |
| 0x2ED | Engine load (alt.) | b5 | raw x 100/255 | % | LIKELY |
| 0x4B2 | Oil temp | b2-b3, 8 bits >> 6 | raw - 40 | °C | CONFIRMED |
| 0x417 | **Intake air temp** | b2-b3, 11 bits >> 5 | raw - 40 | °C | LIKELY |
| 0x41B | **Throttle plate** | b4-b5, 16 bits | raw x 0.03754 + 10.58 | % | LIKELY |
| 0x384 | Outside temp | b1-b2, 9 bits >> 1 | raw - 142 | °F | CONFIRMED |
| 0x384 | **DNA drive mode** | b1 bits 3-4 | (b1 >> 3) & 3: 0 = Natural, 1 = Dynamic, 2 = Advanced | | CONFIRMED |
| 0x0FA | Brake pedal | b0 bits 3-2 | 0 = released, 3 = pressed (b0 0x80 / 0x8C) | | CONFIRMED |
| 0x107 | **Brake pressure** | b0 | raw (0 foot off, 60-70 holding the car still, ~45-52 in a firm stop) | bar? | GUESS |
| 0x1FA | Brake pedal travel? | b0 | raw - 16 (16 released, ~36 pressed) | raw | GUESS |
| 0x256 | Parking brake (EPB) | b0 bit 3 | 1 = applied (b0 0x28 on, 0x20 off) | | LIKELY |
| 0x73E | **Left turn signal** | b1 bit 7 | 1 = lamp on (blinks) | | LIKELY |
| 0x73E | **Right turn signal** | b2 bit 5 | 1 = lamp on (blinks) | | LIKELY |
| 0x416 | VIN | b0 = part 0-2, b1-b7 ASCII | 3 parts = 17 characters | text | CONFIRMED |
| 0x5A6 | Climate: fan in manual (AUTO off) | b0 bit 7 | 1 = manual | | GUESS |
| 0x5A6 | Climate: blower command | b3 | lower = faster (0xB1 lowest ... 0x16 max, 0x3C off) | raw | GUESS |
| 0x5A6 | Climate: mix / outlet target | b4-b5, 10 bits >> 6 | rises with the setpoint above 67 °F | raw | GUESS |
| 0x41E | Battery / IBS? | b0-b1 | jumps at engine start and on every window motor press | raw | GUESS |

How the new ones were checked (drive of 2026-11-17, nothing dropped, reference values asked ~20 times a second):

* **Boost** matches the engine's boost sensor (UDS `22 195A` minus barometric pressure) with r = 0.999 and a
  median error of 5 mbar, all the way to **26 psi** on full-throttle pulls. It reads 0 under vacuum, like a dash gauge.
* **Speed** matches OBD PID 0x0D up to 77 mph with a median error of **0.08 mph**.
* **Pedal** equals OBD PID 0x49 (pedal sensor D). Floored is ~84 %; pushing through the **kickdown switch** reads ~90 %.
  The gear byte shows the kickdown downshift (5th -> 3rd -> 2nd in D, 4th -> 3rd even in manual mode).
* **Steering**: 7200 raw with the wheel straight, 11244 at full left, 3163 at full right = +-404 deg,
  about 2.25 turns lock to lock.
* **Turn signals**: with a front-left bulb out, the left bit blinked twice as fast (every 0.17 s instead of
  0.34 s), the same bulb-out hyper-flash you see on the dash.
* **Intake air** equals the engine's own intake-air value (UDS `22 1935`) one-to-one, r = 0.998.
* **Oil temp** (0x4B2) is within ~2 °C of the engine's oil value (UDS `22 1302`). It's sent about once a second, so it lags.

Found later (at-the-car data-ID scan, engine running, then a short drive):

* **DNA drive mode** (0x384 b1 bits 3-4): recorded while turning the knob through each position; raw b1 reads 1 = Natural, 9 = Dynamic, 17 = Advanced (the mode sits in bits 3-4). A second capture held in Dynamic confirmed the mapping.
* **Oil pressure** (engine UDS `22 1894`): reads 0 with the engine off and climbs smoothly with rpm (r ~ 0.8) — the signature of real oil pressure. The nearby `1893` (target) and `1891` track it. Values are low on purpose: this engine's variable-displacement oil pump holds pressure down for economy (~1.35 bar / 20 psi idle, ~2.4 bar / 35 psi at 3,300 rpm).
* **Transmission fluid temp** (transmission UDS `22 1009`): the only data-ID that climbed monotonically over a drive while the others bounced or pegged; ~85 °C at idle (matches coolant via the trans heat-exchanger), ~110 °C worked.

### Gear (0x2EF b0 high nibble)

| Raw | 0 | 1-6 | 7 | 8 | 9 / A | F |
|---|---|---|---|---|---|---|
| Gear | N | 1st-6th | R | **7th** | **8th** | P |

Checked with RPM / mph in each gear over all drives: 3rd 89, 4th 72, 5th 55, 6th 42, 7th 34, 8th 27, which
matches the ZF 8HP ratios (4.71, 3.14, 2.11, 1.67, 1.29, 1.00, 0.84, 0.67). In manual mode the low nibble
shows the next gear while a shift is prepared (0x34 = in 3rd, going to 4th).

### Messages seen but not decoded yet

0x0EE (all of b0-b5 zero = standing still; b5 follows speed in km/h 1:1, probably a wheel speed),
0x100 (very busy while rolling, probably wheel speeds), 0x0FE (stability: b3-b4 sit at 0x800 parked and move
only while driving, yaw rate or lateral acceleration candidate), 0x419 / 0x41A (body: bursts on window and door
events), 0x192 (b3-b4 count while cluster warnings are on), 0x1EF.

Battery voltage (OBD PID 0x42, 13.8-14.4 V on the drive) has no broadcast match yet.
0x0FE b0-b1 (12 bits >> 4, centered on 2048) follows deceleration while braking and barely moves when parked with the brake held: probably a longitudinal acceleration sensor.

## Asking the engine computer (UDS / OBD-II)

* OBD-II is **29-bit**: functional requests to `18DB33F1`, answers from `18DAF110` (engine) and `18DAF118`
  (transmission). 11-bit `7DF` gets no answer from the engine.
* Supported PIDs include 04 load, 05 coolant, 0B MAP (tops out at 254 kPa under boost), 0C RPM, 0D speed,
  0E timing, 10 MAF, 11 throttle, 2F fuel level, 33 baro, 42 module voltage, 46 ambient, 49 / 4A pedal.
* Alfa-specific values, read with UDS `22 xx xx` sent to `18DA10F1`:

| DID | Value | Format |
|---|---|---|
| 195A | Boost (charge) pressure | 16 bits, **mbar absolute**. ~990 at idle (= outside air), 2,792 at peak. Boost = value - baro |
| 192D | Gear | plain number (3, 4, 5... 8; 0 when stopped in P/N) |
| 1935 | Intake air temperature | 1 byte, value - 40 = °C (read 30-41 °C) |
| 1302 | Oil temperature | 16 bits, °C (read 49-87 °C, matches 0x4B2 within ~2 °C) |
| 1894 | **Oil pressure** (actual) | 16 bits, **mbar gauge**. ~1,350 (1.35 bar / 20 psi) idle, ~2,400 (2.4 bar / 35 psi) at 3,300 rpm, 0 with the engine off. `1893` = target, `1891` = related |

The **transmission** computer (`18DA18F1`) also answers UDS `22`: **`1009` = transmission fluid temp** (16 bits, °C) — ~85 °C idle (tracks coolant via the trans heat-exchanger), ~110 °C worked.

* Modules that answer UDS TesterPresent (`3E 00`) on `18DAxxF1`:

| Request -> answer | Module |
|---|---|
| 18DA10F1 -> 18DAF110 | Engine (ECM) |
| 18DA18F1 -> 18DAF118 | Transmission (TCM), also 11-bit 7E1 -> 7E9 |
| 18DA40F1 -> 18DAF140 | Body computer (BCM)? (most network / body codes are here) |
| 18DA60F1 -> 18DAF160 | Instrument cluster? |
| 18DA1AF1, 18DA1FF1, 18DAC7F1 | Unknown |

Trouble codes are read with UDS `19 02`. The byte after a code (e.g. U1751**-87**) is the standard failure
type: 87 = missing message, 86 = invalid signal, 13 = open circuit, 11 = short to ground, 4A = wrong component
installed, 64 = plausibility failure.

## Checked and rejected

Don't use these, they look right at first but aren't:

* 0x0FA b4 as speed (a body counter, ~6 mph off). Use 0x101.
* 0x100 b2-b3 as speed or coolant (wheel dynamics ticks).
* 0x1F7 b5 as coolant, 0x104 b7 / 0x0FA b5 as oil temp, 0x0FB b1 / 0x192 as outside temp.
* 0x0FC b0 high nibble as engaged gear while moving (it's RPM). Use 0x2EF.
* 0x0DE b4 as throttle (it's the rolling counter). Use 0x412 b3 for the pedal.
* 0x0FB b3 x 0.1 as boost psi (reads 22 psi with the engine off), and 0x0FB b2-b3 as boost target
  (only looked right while revving in Park). Use 0x2EF b3.

## How I recorded it

I used an ESP32 with a CAN transceiver on the OBD-II port, logging in `candump -L` format. For plain recordings
it's in **listen-only** mode. For the reference recordings it also sends standard read requests (OBD mode 01,
UDS 22) about 20 times a second, a tiny share of the bus.
Any CAN logger that writes candump or SavvyCAN format works the same way.
To find a signal: record while doing **one** thing, add a marker the moment you do it, and compare the bytes
before and after. For values the engine computer knows (temperatures, pressures, pedal), ask for them while
recording and look for the broadcast bytes that follow them.

Only ever send standard read requests (OBD mode 01, UDS 22 / 19 / 3E). Don't clear codes or write anything
you don't fully understand.
