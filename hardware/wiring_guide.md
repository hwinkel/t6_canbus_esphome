# VW T6.1 CAN Scanner - Hardware Wiring Guide

## ⚠️ Safety Warning
**ALWAYS disconnect the vehicle battery before working on electrical systems!**
- Work only on the Comfort-CAN bus (NOT drivetrain CAN)
- Use proper crimping tools for automotive connectors
- Test all connections before final installation
- This modification is for research/hobby use only

## 1. Required Components

### 1.1 Main Hardware
| Item | Part Number | Quantity | Notes |
|------|------------|----------|-------|
| LILYGO T-CAN485 | - | 1 | ESP32 + CAN transceiver board |
| 12V to 5V Buck Converter | (Built into T-CAN485) | - | Onboard regulator |
| TVS Diode | P6KE15CA | 1 | Bi-directional, 15V standoff |
| Inline Fuse Holder | - | 1 | Mini blade type |
| Fuse | 2A mini blade | 1 | Protection for 12V line |

### 1.2 Connectors & Cables
| Item | Part Number | Quantity | Notes |
|------|------------|----------|-------|
| MQS Male Pins | N 907 647 01 | 4 | VW/Audi part |
| MQS Female Housing | 1J0 973 702 | 1 | 2-pin connector |
| Wire (Power) | 1.0mm² red/black | 2m | 12V and GND |
| Wire (CAN) | 0.35mm² twisted pair | 1m | CAN-H and CAN-L |
| Heat Shrink Tubing | Various | - | 3:1 ratio recommended |
| Cable Gland | PG7 | 1 | For enclosure entry |

### 1.3 Enclosure
| Item | Specifications | Notes |
|------|---------------|-------|
| ABS Box | 100x60x35mm min | IP54 or better |
| Mounting Tape | 3M VHB | Double-sided automotive |
| Cable Ties | 150mm | For strain relief |

## 2. Pin Connections

### 2.1 LILYGO T-CAN485 Pinout
```
┌─────────────────────────┐
│   LILYGO T-CAN485 V1.2  │
│                         │
│  12V ──○ [DC IN +12V]   │ ← Vehicle 12V (fused)
│  GND ──○ [GND]          │ ← Vehicle ground
│        │                │
│ CAN-H ──○ [CAN-H]       │ ← To vehicle CAN-H
│ CAN-L ──○ [CAN-L]       │ ← To vehicle CAN-L
│        │                │
│        ○ [GPIO 25]      │ ← TX Guard LED (optional)
│        ○ [GPIO 26]      │ ← Status LED (optional)
│                         │
│    [USB-C Port]         │ ← Programming/Debug
│                         │
│    [WiFi Antenna]       │ ← Internal antenna OK
└─────────────────────────┘
```

### 2.2 VW T6.1 Gateway Connector
Location: Driver footwell, behind kickplate

```
Gateway 20-pin Connector (Looking at pins):
┌─────────────────────────────────┐
│  1  2  3  4  5  6  7  8  9  10 │
│ 11 12 13 14 15 16 17 18 19 20  │
└─────────────────────────────────┘

Comfort-CAN Pins:
- Pin 6:  CAN-H (Green/Orange-Green)
- Pin 14: CAN-L (Orange/Brown)
- Pin 20: Ground (Brown)
- Pin 1:  12V+ (Red/White) - Always on
```

## 3. Wiring Diagram

```
                            ┌──────────────┐
    Vehicle 12V+ ──────[2A]─┼─○ 12V       │
           (Pin 1)     Fuse │              │
                           │   ESP32      │
    Vehicle GND ────────────┼─○ GND       │
          (Pin 20)         │              │
                           │   T-CAN485   │
    CAN-H (Pin 6) ─────────┼─○ CAN-H     │
         Green/Orange      │              │
                           │              │
    CAN-L (Pin 14) ────────┼─○ CAN-L     │
         Orange/Brown      └──────────────┘
                           
    [2A]: 2A inline fuse
    ───: Twisted pair for CAN
    ───: Power wires (1.0mm²)
```

## 4. Step-by-Step Installation

### 4.1 Preparation
1. **Disconnect vehicle battery negative terminal**
2. Remove driver footwell kickplate (3 clips)
3. Locate gateway module (silver box with connectors)
4. Identify 20-pin connector (usually gray or black)

### 4.2 Create the Harness
1. **Prepare CAN wires:**
   - Cut 50cm of twisted pair cable
   - Strip 5mm from each end
   - Tin the stripped ends lightly

2. **Prepare power wires:**
   - Cut 75cm red wire (12V+)
   - Cut 75cm black wire (GND)
   - Install inline fuse holder on red wire
   - Strip and tin ends

3. **Crimp MQS pins:**
   - Use proper MQS crimping tool
   - Crimp pins to all 4 wires
   - Pull-test each connection (should hold 5kg)

### 4.3 Tap into Gateway Connector

**Method A: T-Tap (Reversible)**
```
1. Carefully release connector from gateway
2. Use T-tap connectors on wires:
   - Red T-tap on Pin 1 wire (12V)
   - Black T-tap on Pin 20 wire (GND)
   - Green T-tap on Pin 6 wire (CAN-H)
   - Orange T-tap on Pin 14 wire (CAN-L)
3. Connect your harness to T-taps
4. Secure with heat shrink
```

**Method B: Pin Insertion (Cleaner)**
```
1. Remove connector from gateway
2. Carefully release pin lock tab
3. Insert your MQS pins alongside existing:
   - Double-up pins in positions 1, 6, 14, 20
4. Ensure pins lock properly
5. Reattach connector to gateway
```

### 4.4 Connect to ESP32 Board

1. **Inside enclosure:**
   - Mount T-CAN485 with standoffs
   - Connect 12V to DC IN terminal
   - Connect GND to GND terminal
   - Connect CAN-H to CAN-H terminal
   - Connect CAN-L to CAN-L terminal

2. **Add protection:**
   - Install TVS diode across CAN-H and CAN-L
   - Add 120Ω termination resistor if needed (usually not)

3. **Secure connections:**
   - Use ferrules on screw terminals
   - Apply thread lock to terminal screws
   - Use cable gland for harness entry

### 4.5 Final Installation

1. **Test before closing:**
   - Reconnect battery
   - Power on device (LED should light)
   - Check WiFi AP appears
   - Verify no DTCs in vehicle

2. **Mount enclosure:**
   - Clean mounting surface with alcohol
   - Apply VHB tape to enclosure
   - Mount high in footwell (away from feet)
   - Secure harness with cable ties

3. **Cable management:**
   - Route cables away from moving parts
   - Maintain minimum 10cm from airbag modules
   - Keep CAN twisted pair away from power wires
   - Leave service loop for future access

## 5. Electrical Specifications

### 5.1 CAN Bus
- **Protocol:** ISO 11898-2
- **Bitrate:** 500 kbps
- **Voltage:** 2.5V nominal (recessive)
- **Differential:** 2V (dominant)
- **Termination:** 60Ω total (2x 120Ω at bus ends)
- **Stub length:** Maximum 30cm

### 5.2 Power
- **Input:** 12-14.4V DC (vehicle)
- **Current:** 150mA typical, 250mA max
- **Protection:** 2A fuse + TVS diode
- **Regulation:** Onboard 5V/3.3V

## 6. Testing & Validation

### 6.1 Bench Test (Before Installation)
```bash
# Connect via USB to computer
esphome logs t6_can_scanner.yaml

# Expected output:
[I][can:101] CAN bus initialized at 500kbps
[I][can:102] Mode: LISTEN_ONLY
[I][wifi:200] AP Mode: t6-can-setup
```

### 6.2 Vehicle Test
1. **Initial power-on:**
   - Status LED: Solid (boot)
   - Status LED: Blink (WiFi connecting)
   - Status LED: Solid (connected)

2. **CAN verification:**
   - Open web UI: http://t6-can.local
   - Should see frames immediately
   - Typical rate: 50-200 fps idle

3. **Check for errors:**
   - No DTCs in vehicle diagnostic
   - No CAN error frames in log
   - Bus state: "Active"

## 7. Troubleshooting

### 7.1 No CAN Data
- **Check:** CAN-H and CAN-L not swapped
- **Check:** Gateway connector fully seated
- **Check:** Bitrate set to 500kbps
- **Test:** Measure 60Ω between CAN-H and CAN-L (ignition off)

### 7.2 Device Won't Boot
- **Check:** 12V present at board input
- **Check:** Fuse not blown
- **Check:** Ground connection solid
- **Test:** Measure 12V at board terminals

### 7.3 WiFi Connection Issues
- **Check:** Antenna connected (if external)
- **Check:** Correct WiFi credentials
- **Check:** Within range of access point
- **Test:** Use WiFi AP mode for initial setup

### 7.4 Vehicle Errors/DTCs
- **Immediately:** Disconnect device
- **Check:** No shorts in wiring
- **Check:** Using Comfort-CAN not drivetrain
- **Action:** Clear DTCs and test without device

## 8. Wire Color Reference

### 8.1 VW T6.1 Standard Colors
| Function | Wire Color | Pin |
|----------|------------|-----|
| 12V+ Constant | Red/White | 1 |
| Ground | Brown | 20 |
| Comfort CAN-H | Green/Orange-Green | 6 |
| Comfort CAN-L | Orange/Brown | 14 |
| Drivetrain CAN-H | Orange/Black | DO NOT USE |
| Drivetrain CAN-L | Orange/Brown | DO NOT USE |

### 8.2 Harness Colors (Recommended)
| Function | Use Color |
|----------|-----------|
| 12V+ | Red |
| Ground | Black |
| CAN-H | Green or White |
| CAN-L | Brown or White/Brown |

## 9. Safety Checklist

Before closing installation:
- [ ] All connections crimped and tested
- [ ] No bare wires exposed
- [ ] Fuse installed and correct value (2A)
- [ ] TVS diode installed for surge protection
- [ ] CAN stub length < 30cm
- [ ] No interference with pedals or controls
- [ ] Harness secured and strain relieved
- [ ] Device set to LISTEN_ONLY mode
- [ ] No DTCs present in vehicle
- [ ] WiFi connection established

## 10. Pinout Reference Card

Print and keep in vehicle:
```
┌─────────────────────────────┐
│   QUICK REFERENCE CARD      │
├─────────────────────────────┤
│ T-CAN485 │ Wire   │ Gateway │
├──────────┼────────┼─────────┤
│ 12V IN   │ Red    │ Pin 1   │
│ GND      │ Black  │ Pin 20  │
│ CAN-H    │ Green  │ Pin 6   │
│ CAN-L    │ Orange │ Pin 14  │
├─────────────────────────────┤
│ WiFi AP: t6-can-setup       │
│ Web UI:  t6-can.local       │
│ Default: LISTEN_ONLY mode   │
└─────────────────────────────┘
```

---

**Document Version:** 1.0  
**Last Updated:** 2025-08-28  
**⚠️ Disclaimer:** Modifications to vehicle electrical systems may void warranty. Work at your own risk.