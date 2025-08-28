# VW T6.1 CAN ID Sources and Discovery Methods

## ⚠️ Important Disclaimer
The CAN IDs listed in this project are **hypothetical examples** based on common automotive CAN patterns. They need to be discovered and validated for your specific vehicle.

## Sources of CAN IDs

### 1. **Official Sources (Limited)**
- **VW ODIS/VCDS Documentation**: Official VW diagnostic tools sometimes reveal CAN IDs
- **J1939 Standards**: Commercial vehicles often follow SAE J1939 (not directly applicable to T6.1 Comfort CAN)
- **OBD-II PIDs**: Standard diagnostic IDs (mostly on drivetrain CAN, not comfort)

### 2. **Community Sources**
- **T6 Forum (www.t6forum.com)**: UK-based forum with technical discussions
- **VWCalifornia.com**: California-specific community
- **GitHub Projects**:
  - Similar projects for T5/T6 models may have partial mappings
  - OpenDBC project (though focused on driver assistance)
- **Ross-Tech Forums**: VCDS users sometimes share findings

### 3. **Reverse Engineering Methods**

#### A. Passive Discovery (Recommended First Step)
```yaml
# ESPHome config for logging ALL frames
canbus:
  - platform: esp32_can
    on_frame:
      - lambda: |-
          ESP_LOGD("can_raw", "ID: 0x%03X, Data: %02X %02X %02X %02X %02X %02X %02X %02X",
                   x.can_id,
                   x[0], x[1], x[2], x[3], x[4], x[5], x[6], x[7]);
```

Then:
1. **Log baseline**: Vehicle off, everything idle
2. **Change one thing**: Turn on interior lights
3. **Compare logs**: Find which IDs/bytes changed
4. **Repeat**: For each function you want to decode

#### B. Pattern Recognition
Common CAN ID ranges in VW vehicles:
- `0x000-0x0FF`: High priority/safety (usually not on Comfort CAN)
- `0x100-0x2FF`: Powertrain (engine, transmission)
- `0x300-0x3FF`: Chassis and body control
- `0x400-0x4FF`: Comfort features (likely range for T6.1)
- `0x500-0x5FF`: Infotainment and auxiliary
- `0x600-0x6FF`: Diagnostics
- `0x700-0x7FF`: Manufacturer specific

#### C. Correlation Method
1. **Use multimeter**: Measure actual values (battery voltage, temperature)
2. **Search CAN data**: Look for matching values in different encodings
3. **Common encodings**:
   - Temperature: Often (value * 0.5) - 40 for °C
   - Voltage: Often in mV (value / 1000)
   - Percentage: value * 100 / 255
   - Speed: Often in 0.01 km/h units

### 4. **Typical VW T6 CAN Messages (To Be Validated)**

Based on other VW models and educated guesses:

| CAN ID | Likely Function | Discovery Method |
|--------|----------------|------------------|
| 0x1A0-0x1AF | Vehicle speed, RPM | Common VW range |
| 0x280-0x288 | Steering angle, ABS | Standard chassis |
| 0x300-0x308 | Door status | Body control module |
| 0x380-0x388 | Light status | BCM lighting |
| 0x3B0-0x3BF | Interior lighting | Comfort lighting |
| 0x420-0x428 | Climate control | HVAC standard |
| 0x470-0x478 | Parking sensors | PDC if fitted |
| 0x500-0x510 | Radio/Navigation | Infotainment |
| 0x5A0-0x5A8 | Auxiliary equipment | Fridge common range |
| 0x600-0x6F0 | Diagnostic (UDS) | ISO 14229 |
| 0x670-0x678 | Battery management | Auxiliary systems |

### 5. **Dometic Fridge CAN (CFX3 Series)**
Dometic fridges may use their own protocol:
- Often on 0x5A0-0x5AF range
- May use proprietary encoding
- Check Dometic CI-BUS documentation

### 6. **Discovery Tools**

#### Linux (with CAN adapter):
```bash
# Dump all CAN traffic
candump can0

# Filter specific ID
candump can0,0x3B0:0x7FF

# Log to file
candump -l can0
```

#### Python script for analysis:
```python
import can

bus = can.interface.Bus(bustype='socketcan', channel='can0', bitrate=500000)

# Track changing values
last_data = {}

for msg in bus:
    if msg.arbitration_id not in last_data or last_data[msg.arbitration_id] != msg.data:
        print(f"0x{msg.arbitration_id:03X}: {msg.data.hex()}")
        last_data[msg.arbitration_id] = msg.data
```

### 7. **Safety During Discovery**

**NEVER blindly send CAN messages!**
- Always start in LISTEN_ONLY mode
- Document everything you discover
- Test TX only with known safe messages
- Have emergency stop ready (pull fuse)
- Don't experiment while driving

### 8. **Validation Process**

1. **Hypothesis**: "0x3B0 byte 3 might be LED brightness"
2. **Test**: Change lights manually, observe byte 3
3. **Confirm**: Values correlate (0=off, 255=max)
4. **Scale**: Determine scaling factor
5. **Document**: Record findings with test conditions

### 9. **Common Data Encodings**

| Type | Common Encodings |
|------|------------------|
| Temperature | `(raw * 0.5) - 40` or `(raw * 0.1) - 273.15` (Kelvin) |
| Voltage | `raw / 1000` (mV to V) or `raw * 0.1` |
| Current | `(raw - 32768) * 0.1` (centered at 0) |
| Speed | `raw * 0.01` (km/h) or `raw / 256` |
| Percentage | `raw * 100 / 255` or direct 0-100 |
| Boolean | Bit flags in single byte |
| RPM | `raw * 0.25` or `raw * 4` |

### 10. **Community Contribution**

Once you discover and validate CAN IDs:

1. **Test thoroughly** in your vehicle
2. **Document** the discovery method
3. **Include** vehicle details (year, model, options)
4. **Submit** via GitHub PR to help others
5. **Format** as YAML decoder rules

## Example Discovery Session

```yaml
# 1. Start with logging everything
# 2. Turn on interior lights
# Found: 0x3B0 changes from 00 00 00 00 to 00 00 01 FF
# 3. Dim lights to 50%
# Found: 0x3B0 shows 00 00 01 80 (0x80 = 128 = 50% of 255)
# 4. Conclusion: 0x3B0 byte 2 = on/off, byte 3 = brightness

- can_id: 0x3B0
  name: "interior_lights"
  byte_2_bit_0: lights_on_off
  byte_3: brightness_0_255
```

## Resources

- [SavvyCAN](https://www.savvycan.com/) - CAN bus reverse engineering tool
- [CANBadger](https://github.com/Gutenshit/CANBadger) - CAN analysis toolkit  
- [can-utils](https://github.com/linux-can/can-utils) - Linux CAN utilities
- [Python-CAN](https://python-can.readthedocs.io/) - Python CAN library

---

**Remember**: The IDs in our decoder_rules are EXAMPLES. You must discover and validate the actual IDs for your specific T6.1 vehicle through careful testing and observation.