# ESP8266 Kitchen Control System

A smart kitchen control system using ESP8266 D1 Mini to manage an induction cooktop, water heater, and water pump with physical button controls, RGB status indicators, and Home Assistant integration.

## 🔧 Components Required

### Microcontroller
- **ESP8266 D1 Mini** - Main controller board
  - 11 digital I/O pins
  - WiFi capability
  - 3.3V logic level
  - Built-in voltage regulator (5V input)

### Relay Modules
- **1x Single Channel 5V Relay Module** (NOYITO or equivalent)
  - For Induction Cooktop control
  - 30A @ 250V AC capacity
  - Optocoupler isolation
  - Active LOW trigger (3.3V compatible)

- **1x Dual Channel 5V Relay Module** (NOYITO or equivalent)
  - For Water Heater and Water Pump control
  - 30A @ 250V AC per channel capacity
  - Optocoupler isolation
  - Active LOW trigger (3.3V compatible)

### Indicators & Controls
- **2x WS2812B RGB LED Pixels** (NeoPixel compatible)
  - Addressable RGB LEDs
  - 5V power, 3.3V data compatible
  - Chainable on single data line

- **2x Momentary Push Buttons**
  - Standard tactile switches
  - NO (Normally Open) configuration
  - Used with internal pull-up resistors

### Power Supply
- **5V DC Power Supply** (minimum 1A capacity)
  - Powers ESP8266, relay modules, and RGB LEDs
  - Total current draw: ~500mA maximum

## 📋 Pin Assignments

| Function | ESP8266 Pin | GPIO | Notes |
|----------|-------------|------|-------|
| **Induction Cooktop Relay** | D1 | GPIO5 | Single relay module trigger |
| **Water Heater Relay** | D2 | GPIO4 | Dual relay CH1 trigger |
| **Water Pump Relay** | D5 | GPIO14 | Dual relay CH2 trigger |
| **RGB Pixels Data** | D6 | GPIO12 | WS2812B data line |
| **Control Button 1** | D7 | GPIO13 | Main control (cycle states) |
| **Control Button 2** | D0 | GPIO16 | Water pump toggle |
| **Power Input** | 5V | - | From 5V power supply |
| **Ground** | GND | - | Common ground |

### Pin Selection Rationale
- **D1, D2, D5, D6, D7**: Safe GPIO pins with no special boot behavior
- **D0**: Safe for button input (doesn't affect boot when using pull-up)
- **Avoided**: D3, D4, D8, RX, TX (these have special boot requirements or debugging functions)

## 🔌 Wiring Diagram

```
ESP8266 D1 Mini Connections:
┌─────────────────────┐
│     ESP8266 D1      │
│       Mini          │
├─────────────────────┤
│ 5V  ──┬─────────────┼─── 5V Power Supply (+)
│       ├─ Single Relay (5V+)
│       ├─ Dual Relay (5V+)  
│       └─ RGB LEDs (5V+)
│                     │
│ GND ──┬─────────────┼─── Power Supply (-)
│       ├─ Single Relay (GND)
│       ├─ Dual Relay (GND)
│       ├─ RGB LEDs (GND)
│       ├─ Button 1 (one side)
│       └─ Button 2 (one side)
│                     │
│ D1  ─────────────────┼─── Single Relay (Trigger)
│ D2  ─────────────────┼─── Dual Relay (CH1 Trigger)
│ D5  ─────────────────┼─── Dual Relay (CH2 Trigger)  
│ D6  ─────────────────┼─── RGB LEDs (Data In)
│ D7  ─────────────────┼─── Button 1 (other side)
│ D0  ─────────────────┼─── Button 2 (other side)
└─────────────────────┘
```

### Relay Module Connections
Each relay module has the following terminals:
- **5V+, GND**: Power input (from 5V supply)
- **Trigger**: Control signal (from ESP8266 GPIO)
- **COM, NO, NC**: High-voltage switching contacts for your appliances

## ⚡ Power Requirements

| Component | Voltage | Current Draw | Notes |
|-----------|---------|--------------|-------|
| ESP8266 D1 Mini | 3.3V (via 5V input) | ~80mA | Internal regulator |
| Single Relay Module | 5V | ~5mA idle, 190mA active | Per module |
| Dual Relay Module | 5V | ~5mA idle, 380mA active | Both channels |
| RGB LEDs (2x) | 5V | ~60mA max each | When at full brightness |
| **Total System** | **5V** | **~500mA max** | Recommended 1A supply |

## 🎮 Control Functions

### Button 1 - Main Control (3-State Cycle)
1. **First Press**: Turn ON Induction Cooktop
2. **Second Press**: Turn OFF Induction → Turn ON Water Heater (45-min timer)
3. **Third Press**: Turn OFF Water Heater → Return to OFF state

### Button 2 - Water Pump
- **Press**: Toggle Water Pump ON/OFF
- **Auto-Off**: 1-hour timer when activated

### Safety Features
- **Mutual Exclusion**: Induction Cooktop and Water Heater cannot both be active
- **Auto-Timers**: Water Heater (45 min), Water Pump (1 hour)
- **Debouncing**: 200ms button debounce prevents false triggers

## 💡 LED Status Indicators

### Pixel 1 - System State
| State | Color | Pattern |
|-------|-------|---------|
| System OFF | Black | Solid (off) |
| Induction Cooktop | Red | Solid |
| Water Heater | Purple/Orange | Oscillating |

### Pixel 2 - Water Pump
| State | Color | Pattern |
|-------|-------|---------|
| Pump OFF | Black | Solid (off) |
| Pump ON | Blue/Green | Slow pulse |

## 🏠 Home Assistant Integration

The system provides HTTP REST API endpoints for remote control:

### Status Endpoint
```http
GET http://[ESP_IP]/status
```
Returns JSON with current system state:
```json
{
  "induction": true,
  "water_heater": false,
  "water_pump": true,
  "state": 1
}
```

### Control Endpoints
| Endpoint | Method | Function |
|----------|--------|----------|
| `/induction/on` | POST | Turn ON induction cooktop |
| `/induction/off` | POST | Turn OFF induction cooktop |
| `/heater/on` | POST | Turn ON water heater (45min timer) |
| `/heater/off` | POST | Turn OFF water heater |
| `/pump/on` | POST | Turn ON water pump (1hr timer) |
| `/pump/off` | POST | Turn OFF water pump |

## 📚 Required Arduino Libraries

Install these libraries in Arduino IDE:

1. **ESP8266WiFi** (built-in with ESP8266 board package)
2. **ESP8266WebServer** (built-in with ESP8266 board package)  
3. **FastLED** (install via Library Manager)
   ```
   Tools → Manage Libraries → Search "FastLED" → Install
   ```

## 🚀 Setup Instructions

### 1. Hardware Assembly
1. Connect all components according to the wiring diagram
2. Ensure proper power supply connections
3. Double-check relay module trigger connections
4. Test button connections with multimeter if needed

### 2. Software Configuration
1. Open `kitchen_control.ino` in Arduino IDE
2. Update WiFi credentials:
   ```cpp
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
   ```
3. Upload code to ESP8266 D1 Mini

### 3. Testing
1. Open Serial Monitor (115200 baud) to view system status
2. Test each button function
3. Verify LED indicators work correctly
4. Test Home Assistant endpoints using web browser or curl

## ⚠️ Safety Warnings

- **High Voltage**: Relay contacts switch mains voltage - ensure proper electrical installation
- **Load Ratings**: Do not exceed relay capacity (30A @ 250V AC)
- **Isolation**: Relay modules provide optocoupler isolation for safety
- **Professional Installation**: Have a qualified electrician wire high-voltage connections
- **Power Supply**: Use appropriate 5V DC supply with sufficient current capacity

## 🔧 Troubleshooting

### System Won't Start
- Check 5V power supply connection
- Verify ESP8266 power LED is on
- Check Serial Monitor for boot messages

### Buttons Not Responding  
- Verify button wiring (one side to GPIO, other to GND)
- Check internal pull-up resistor configuration in code
- Ensure 200ms debounce isn't causing issues

### Relays Not Switching
- Confirm relay trigger voltages (should be 3.3V from ESP8266)
- Check relay module power supply (5V)
- Test relay modules independently
- Verify active-LOW trigger configuration

### WiFi Connection Issues
- Double-check SSID and password
- Ensure 2.4GHz WiFi network (ESP8266 doesn't support 5GHz)
- Check router compatibility with ESP8266

### RGB LEDs Not Working
- Verify 5V power to LED strip
- Check data line connection (D6 → Data In)
- Ensure FastLED library is correctly installed
- Test with simple LED example first

## 📝 License

This project is open source. Feel free to modify and distribute as needed.

## 🤝 Contributing

Contributions welcome! Please submit pull requests or issues on GitHub.

---
**Created for ESP8266 D1 Mini with NOYITO Relay Modules**  
*Version 1.0 - Kitchen Control System*
