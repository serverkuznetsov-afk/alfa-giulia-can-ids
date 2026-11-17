# Recordings

All files are `candump -L` text, gzip-compressed (`.log.gz`):
```
(1783574473.447000) can0 0FC#0BB4C002183FD289
# MARK 1783574473.000 LEFT FRONT DOWN          <- what was done at that moment
```
They open in SavvyCAN, python-can (`can.LogReader`), cantools, `canplayer`, or with plain `zcat`.

* **`old_firmware/`**: I recorded these with an older firmware that dropped about **90 %** of frames. A 100 Hz ID like 0x0FC
  arrives at ~7 Hz. What did arrive is intact (CRC-8 checks pass), so values are right, just sparse.
* **`obd_reference/`**: recorded with my current firmware, nothing dropped (~2,500 frames/s). While recording,
  my PC asked the engine computer for known values about 12 times a second. The requests (`18DB33F1`, `18DA10F1`) and answers
  (`18DAF110`) are in the log, which is how broadcast signals get matched to real values.
* The VIN (ID 0x416) has been replaced with `X` in every file.
* Times are Unix epoch seconds; the car was in US Central time (UTC-5).

| File | Length | Frames | Marks | What was done |
|---|---|---|---|---|
| `old_firmware/raw_can_20260708_205449.log.gz` | 3 s | 304 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_205451.log.gz` | 4 s | 447 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_205553.log.gz` | 3 s | 287 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_205601.log.gz` | 3 s | 382 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_205610.log.gz` | 3 s | 300 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_205616.log.gz` | 4 s | 441 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_205700.log.gz` | 3 s | 344 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_205701.log.gz` | 4 s | 409 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_213009.log.gz` | 5 s | 567 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_213243.log.gz` | 1 s | 86 |  | Short raw clip (engine on, parked) |
| `old_firmware/raw_can_20260708_214050.log.gz` | 1 s | 124 |  | Short raw clip (engine on, parked) |
| `old_firmware/session_20260709_001547_Engine_start_with_brake.log.gz` | 10 s | 1,108 |  | Engine start with foot on the brake |
| `old_firmware/session_20260709_001628_Ebrake_disengage_engage_with_brake.log.gz` | 12 s | 1,383 |  | Parking brake off / on with foot on the brake |
| `old_firmware/session_20260709_002054_Driver_door_panel_for_windows.log.gz` | 118 s | 13,168 | 8 | All four windows down / up from the driver door panel |
| `old_firmware/session_20260709_002538_Ebrake_disengage_engage.log.gz` | 80 s | 8,936 | 4 | Parking brake off / on, with and without the brake pedal |
| `old_firmware/session_20260709_004010.log.gz` | 2 s | 237 |  | Short idle clip |
| `old_firmware/session_20260709_004022_gas_padel.log.gz` | 61 s | 6,756 | 2 | Full throttle stab, then holding 3500 rpm (Park) |
| `old_firmware/session_20260709_004132_gear_box.log.gz` | 258 s | 28,627 | 10 | Shifting P-D-R-N-D-M, M1/M2 paddles, back to P (stopped, foot on brake) |
| `old_firmware/session_20260709_005123.log.gz` | 61 s | 6,816 | 10 | Parking brake engage / release x5 |
| `old_firmware/session_20260709_005655_ebrake_parked_brake_engine_on.log.gz` | 113 s | 12,591 | 10 | Parking brake engage / release x5, engine on |
| `old_firmware/session_20260709_010816_ebrake_parked_brake.log.gz` | 161 s | 17,926 | 10 | Parking brake engage / release x5, parked (cleanest e-brake session) |
| `old_firmware/session_20260709_011821.log.gz` | 67 s | 7,459 |  | Idle / parked |
| `old_firmware/session_20260709_011949_going_for_a_short_drive.log.gz` | 524 s | 57,272 | 7 | Short drive (39-50 mph), then engine off, lights off, door open, seat moved |
| `old_firmware/session_20260709_060357_manual_drive.log.gz` | 674 s | 75,162 | 35 | Remote start, cold start, drive in manual mode through gears 1-6 with speed/RPM markers |
| `old_firmware/session_20260709_214341.log.gz` | 340 s | 37,838 | 11 | Drive in D with speed markers 30-55 mph |
| `old_firmware/session_20260710_060708_temps.log.gz` | 1,622 s | 179,401 | 65 | Warm idle in Park: outside temp, coolant, fuel; HVAC setpoint 61-84 F step by step; fan speed steps |
| `obd_reference/idle_boost_reference_test.log.gz` | 30 s | 74,819 |  | 30 s idle while polling MAP, baro, RPM, throttle, load and boost (UDS 195A). No frames lost. |
| `obd_reference/idle_then_revving_boost_reference.log.gz` | 615 s | 1,511,807 |  | ~10 min idle, then revving to ~3400 rpm in Park, polling the same values. No frames lost. |
