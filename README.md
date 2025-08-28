# VW T6.1 CAN Scanner - ESPHome Project

Simple ESP32-based CAN bus monitor for VW T6.1 California, integrated with Home Assistant.

## 🚗 What It Does

- **Monitors** the Comfort-CAN bus for vehicle data
- **Decodes** CAN messages into meaningful values (lights, fridge, battery, etc.)
- **Integrates** seamlessly with Home Assistant
- **Provides** web interface for live CAN monitoring
- **Supports** optional control functions (with safety guards)

## ⚡ Quick Start

### 1. Hardware Setup
- Get a **LILYGO T-CAN485** board (ESP32 + CAN transceiver)
- Follow the [wiring guide](hardware/wiring_guide.md) for vehicle connection
- **Safety first**: Always start in LISTEN_ONLY mode!

### 2. ESPHome Configuration
```bash
# Install ESPHome
pip install esphome

# Copy and edit secrets
cp esphome/secrets.yaml.example esphome/secrets.yaml
# Edit with your WiFi credentials

# Flash the firmware
esphome run esphome/t6_can_scanner.yaml
```

### 3. Discovery Mode
The device will log all CAN traffic. Use this to discover your vehicle's specific CAN IDs:

```yaml
# In ESPHome logs, you'll see:
[D][canbus:023] received CAN message (0x3B0): 00 00 01 FF 00 00 00 00
```

See [CAN ID Sources Guide](docs/can_id_sources.md) for discovery methods.

## 📁 Project Structure

```
t6_canbus_esphome/
├── README.md                     # This file
├── docs/
│   ├── architecture.md           # Technical architecture
│   ├── can_id_sources.md         # How to find CAN IDs
│   └── wiring_guide.md           # Hardware installation
├── esphome/
│   ├── t6_can_scanner.yaml       # Main ESPHome config
│   └── secrets.yaml              # WiFi/API credentials
├── decoder_rules/
│   └── vw_t6_comfort.yaml        # Example CAN mappings
└── hardware/
    └── wiring_guide.md            # Connection instructions
```

## 🛡️ Safety Features

- **LISTEN_ONLY by default** - No risk to vehicle systems
- **TX Guard switch** - Must be explicitly enabled for control
- **Whitelist validation** - Only known-safe CAN IDs for transmission
- **Comfort-CAN only** - Isolated from critical drivetrain systems

## 📊 Example Home Assistant Integration

Once configured, you'll get entities like:
- `sensor.interior_brightness` - LED brightness level
- `sensor.fridge_temperature` - Current fridge temp
- `sensor.auxiliary_battery_voltage` - 12V battery status
- `binary_sensor.interior_lights` - Lights on/off state
- `switch.can_tx_enable` - Safety control for transmission

## ⚠️ Important Notes

1. **CAN IDs are examples** - You must discover your vehicle's actual IDs
2. **Test carefully** - Start in LISTEN_ONLY mode, validate thoroughly
3. **Vehicle-specific** - T6.1 California 2024, may vary for other models
4. **Community project** - Share your discoveries to help others!

## 🔧 Advanced Usage

### Custom CAN Message Handling
```yaml
canbus:
  - platform: esp32_can
    on_frame:
    - can_id: 0x123  # Your discovered ID
      then:
        - lambda: |-
            // Your custom decoding logic
            float value = (x[0] << 8 | x[1]) * 0.1;
            id(my_sensor).publish_state(value);
```

### Control Example (TX Armed Mode)
```yaml
number:
  - platform: template
    name: "Set LED Brightness"
    on_value:
      - if:
          condition: switch.is_on: can_tx_enable
          then:
            - canbus.send:
                can_id: 0x3B0
                data: [0x00, 0x00, 0x01, !lambda "return (uint8_t)(x * 2.55);"]
```

## 📚 Resources

- [ESPHome CAN Bus Documentation](https://esphome.io/components/canbus.html)
- [VW T6 Forum](https://www.t6forum.com/) - Community discussions
- [SavvyCAN](https://www.savvycan.com/) - CAN analysis tool

## 🤝 Contributing

Found CAN IDs for your T6.1? Please contribute!

1. Validate IDs thoroughly in your vehicle
2. Document discovery method and vehicle details
3. Submit via GitHub PR
4. Help build the community database

## ⚖️ Legal

- **Research/hobby use only**
- **No warranty** - use at your own risk
- **Respect local laws** regarding vehicle modifications
- **CAN bus interference** can cause serious issues - be careful!

## 📜 License

MIT License - See LICENSE file for details

---

**🚨 Always prioritize safety over features! 🚨**