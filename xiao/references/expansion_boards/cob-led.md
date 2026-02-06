# COB LED Driver Board for XIAO

## Overview

The COB (Chip-on-Board) LED Driver Board for XIAO is designed to control COB LED lighting modules. COB LEDs provide high luminosity with excellent color mixing and uniform illumination. This driver board enables precise control of COB LED brightness and color for applications requiring high-quality lighting solutions.

## Hardware

### Board Compatibility

| XIAO Board | Compatible | Notes |
|------------|------------|-------|
| ESP32C3 | ✅ | Full support |
| ESP32C5 | ✅ | Full support |
| ESP32C6 | ✅ | Full support |
| ESP32S3 | ✅ | Full support |
| nRF52840 | ✅ | Full support |
| RP2040 | ✅ | Full support |
| RP2350 | ✅ | Full support |
| SAMD21 | ✅ | Full support |
| RA4M1 | ✅ | Full support |
| MG24 | ✅ | Full support |
| nRF54L15 | ✅ | Full support |

### Features

- **COB LED Support**: High-brightness COB LED modules
- **Brightness Control**: PWM-based dimming
- **Color Mixing**: RGB COB for full color output
- **Power Management**: Handles high COB LED currents
- **Heat Dissipation**: Designed for COB thermal requirements

### Specifications

| Item | Value |
|------|-------|
| LED Type | COB (Chip-on-Board) |
| Interface | PWM or digital control |
| Power | External power required |
| Current | High (1A+ typical) |
| Voltage | 12V or 24V (depends on COB module) |

### Pin Connections

Pin connections vary by specific COB LED driver board design. Common configurations:

| Function | XIAO Pin | Notes |
|----------|-----------|-------|
| PWM Red | D0 or D1 | PWM output |
| PWM Green | D2 or D3 | PWM output |
| PWM Blue | D6 or D7 | PWM output |
| Enable | D8 or similar | On/off control |

**Important:** COB LEDs require significant current (1A+). Always use external power supply with adequate capacity.

## Platform Implementations

For platform-specific code examples and library information, see:

- **Arduino**: `/xiao-arduino/references/expansion-boards/cob-led.md`
  - PWM control setup
  - Color mixing examples
  - Brightness control
  - Thermal management

- **MicroPython**: Coming soon to `/xiao-micropython/references/expansion-boards/`

## COB LED Advantages

**Compared to Traditional LEDs:**
- Higher luminosity per area
- Better color mixing
- More uniform illumination
- Compact size
- Better thermal performance

## Power Requirements

**Critical:** COB LEDs require significant power:

| COB Size | Current | Voltage |
|----------|---------|---------|
| Small | ~0.5A | 12V |
| Medium | ~1A | 12V or 24V |
| Large | ~2A+ | 24V or 36V |

**Power Supply:**
- Use external power supply adequate for COB module
- Connect power directly to COB LED
- DO NOT power through XIAO

## Usage Notes

1. **External Power:** Required (1A+)
2. **Heat Dissipation:** COB LEDs generate heat - provide heat sink
3. **PWM Frequency:** Use appropriate PWM for COB (500Hz-1kHz)
4. **Current Limiting:** Built-in or external driver required
5. **Color Calibration:** May need adjustment for white balance

## Applications

- **Photography Lighting:** Studio and product photography
- **Video Lighting:** Content creation, streaming
- **Architectural Lighting:** Accent and task lighting
- **Automotive Lighting:** Interior and exterior
- **Grow Lights:** Plant cultivation
- **Stage Lighting:** Performance lighting
- **Retail Display:** Product highlighting

## Troubleshooting

### COB Not Lighting

**Symptom:** COB LED remains dark

**Possible Causes:**
1. No power - Check external power supply
2. Wrong PWM pins - Verify pin connections
3. Enable not set - Check enable pin
4. Driver not configured - Initialize PWM correctly

**Solution:** Verify power supply, check pin connections, enable output

### Poor Color Mixing

**Symptom:** Uneven color distribution

**Possible Causes:**
1. Wrong PWM values - Calibrate RGB balance
2. Low PWM frequency - Increase PWM frequency
3. COB quality - Use quality COB module

**Solution:** Adjust RGB values, increase PWM frequency, verify COB quality

### COB Overheating

**Symptom:** COB gets very hot

**Possible Causes:**
1. No heat sink - Add heat sink
2. Overdriven - Reduce current/brightness
3. Poor ventilation - Improve airflow

**Solution:** Add adequate heat sinking, reduce drive level, improve cooling

### Flickering

**Symptom:** Noticeable flicker

**Possible Causes:**
1. PWM frequency too low - Increase to 500Hz+
2. Duty cycle issues - Check PWM settings
3. Power supply issue - Verify stable power

**Solution:** Increase PWM frequency, check PWM configuration, verify power supply

## COB LED Types

**Single Color:**
- White (various color temperatures)
- Warm White (2700-3000K)
- Cool White (4000-6500K)

**RGB COB:**
- Full color mixing
- Adjustable white balance
- Wide color gamut

## Comparison: COB vs SMD LEDs

| Feature | COB LED | SMD LED |
|---------|---------|---------|
| Brightness | Higher | Lower |
| Color Mixing | Better | Discrete points |
| Form Factor | Compact module | Individual components |
| Thermal | Needs heatsink | Self-cooling |
| Cost | Higher | Lower |
| Application | Pro lighting | General purpose |

**Choose COB if:** High brightness, uniform illumination needed
**Choose SMD if:** Lower cost, discrete LED points acceptable

## Safety Considerations

1. **Heat:** COB LEDs get HOT - provide adequate heatsinking
2. **Current:** High currents - use appropriate wire gauge
3. **Voltage:** Match power supply to COB requirements
4. **Eye Safety:** Bright COB can damage eyes - avoid direct viewing
5. **Power Off:** Allow cooling before handling
