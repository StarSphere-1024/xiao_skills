# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains Claude Code Skills for SeeedStudio XIAO series microcontroller boards. Skills are modular packages that extend Claude's capabilities with specialized knowledge for embedded development.

## Project Structure

```
XIAO_SKILL/
├── xiao/              # Core skill (pin definitions, board selection)
├── xiao-arduino/      # Arduino platform skill
├── xiao-micropython/  # MicroPython platform skill
├── Ref/               # Reference documentation from SeeedStudio
│   ├── PinOut/        # Pinout reference for all XIAO boards
│   ├── SeeedStudio_XIAO/  # Board-specific documentation
│   └── FAQ/           # Troubleshooting guides
└── XIAO_SKILL_DESIGN.md  # Complete project design documentation
```

## Skill Architecture

This project uses a **layered skill architecture** following Progressive Disclosure principles:

1. **`/xiao` (Core)**: Board selection, pin definitions, links to platform-specific skills
2. **Platform Skills** (e.g., `/xiao-arduino`): Complete development guides for each platform
3. **Reference Files**: Detailed documentation loaded on-demand via SKILL.md

When a user asks about XIAO development:
1. Load `/xiao` to identify the board model
2. Load the appropriate platform skill (e.g., `/xiao-arduino`)
3. Read specific reference files as needed

## Creating or Modifying Skills

### Initialization (New Skills Only)

```bash
python %USERPROFILE%\.claude\skills\skill-creator\scripts\init_skill.py <skill-name> --path .
```

### Editing Skills

1. **SKILL.md**: Always read before editing (Edit tool requires prior Read)
2. **Reference files**: Can be created/edited directly
3. **Bundled resources**:
   - `scripts/` - Executable code (Python/Bash)
   - `references/` - Documentation loaded as needed
   - `assets/` - Files used in output (templates, etc.)

### Packaging Skills

After modifying any skill, package it:

```bash
python %USERPROFILE%\.claude\skills\skill-creator\scripts\package_skill.py <skill-folder> [output-dir]
```

This validates and creates a `.skill` file (zip archive).

## Critical Constraints

### Board Platform Support

Before creating code for a board, verify platform support:

| Board         | Arduino | MicroPython | Notes                                           |
| ------------- | ------- | ----------- | ----------------------------------------------- |
| nRF54L15      | ❌       | ✅           | No Arduino support - do not create Arduino code |
| ESP32C3/C6/S3 | ✅       | ✅           | Full support                                    |
| nRF52840      | ✅       | ✅           | Full support                                    |
| RP2040/RP2350 | ✅       | ✅           | Full support                                    |
| SAMD21        | ✅       | ✅           | Arduino                                   |
| RA4M1         | ✅       | ✅           | Arduino UNO R4 chipset                          |
| MG24          | ✅       | ✅           | Matter/Zigbee, Arduino confirmed                |

Always check `Ref/PinOut/XIAO_Pinout_Reference.md` to verify "Arduino" column exists for a board.

### Pin Definition Conflicts

- ESP32C3 D6, D8, D9 have special boot behaviors - always check board reference
- RP2040 requires BOOTSEL mode for firmware upload
- nRF52840 uses double-click RESET for bootloader entry

## File Organization Patterns

### Reference File Structure

```
skill-name/
├── SKILL.md              # Required: YAML metadata + navigation
└── references/
    ├── getting-started/ # Board-specific setup guides
    ├── api/              # Chip-specific APIs (WiFi, BLE, sleep, etc.)
    ├── libraries/        # Common libraries (Wire, SPI, Servo, etc.)
    └── examples/         # Complete project examples
```

### SKILL.md Format

```yaml
---
name: skill-name
description: Brief what it does. Include "when to use" info here.
---

# Skill content (keep <5k words, <500 lines)

## Navigation
- **Board X**: See `getting-started/board-x.md`
- **Feature Y**: See `api/feature-y.md`
```

### Reference File Format

```markdown
# Title

## Overview
Brief description.

## Code Examples
```cpp
// Code with comments
```

## Troubleshooting
### Issue
Solution.
```

## Known Issues & Solutions

1. **nRF54L15 Arduino Support**: Removed - board does not support Arduino (confirmed by user, no Arduino column in reference docs)

2. **ESP32C5 Existence**: Not in `Ref/PinOut/XIAO_Pinout_Reference.md` but user confirmed to keep (ESP32C5 is newer variant of ESP32C3)

3. **MG24 Arduino Support**: Reference docs show empty Arduino column, but user confirmed Arduino support exists

4. **RA4M1 Arduino Support**: Empty Arduino column in reference, but this is the chip used in Arduino UNO R4 - Arduino support confirmed

## Source of Truth

**`Ref/PinOut/XIAO_Pinout_Reference.md`** is the authoritative source for:
- Board existence
- Pin mappings and functions
- Platform support (check Arduino/ESP32/etc. columns)

When in doubt about board specifications or platform support, consult this file first.

## Information Verification Guidelines

**CRITICAL: Always verify information from multiple sources before making changes.** This prevents errors and "hallucinations" that can lead to incorrect documentation or code.

### Verification Process

When verifying board specifications, pin mappings, or capabilities:

1. **Check CSV Reference First**
   - `Ref/PinOut/XIAO_Pinout_Reference.csv` - Primary reference for pin mappings
   - This provides the baseline XIAO unified standard

2. **Verify Against Wiki Documentation**
   - `Ref/SeeedStudio_XIAO/` folder contains board-specific wiki docs
   - Wiki docs often contain additional details not in the CSV (e.g., alternate pin functions, multiple I2C buses)
   - Example: RA4M1 wiki shows 3 I2C buses (D4/D5, D6/D7, D15/D16) while CSV only shows default D4/D5

3. **Cross-Reference Multiple Sources**
   - If CSV and wiki conflict, investigate further
   - Some boards have capabilities not fully documented in either source
   - When uncertain, flag the issue for user clarification

### Common Verification Pitfalls

| Pitfall | Example | Solution |
|---------|---------|----------|
| **Relying on web searches** | ESP32C3 I2C pins from search results showed D0/D1 (wrong) | Always use CSV/wiki docs first |
| **Assuming single source is complete** | RA4M1 CSV shows D4/D5 I2C, but wiki reveals 3 I2C buses | Check wiki for alternate functions |
| **Making assumptions** | Assuming SAMD21 uses different pins than standard | Verify against XIAO unified standard |
| **Propagating errors** | Copying incorrect info from one file to another | Always verify from primary sources |

### When to Use Web Search

Web search should be used **only** for:
- General API documentation (Arduino library docs, chip datasheets)
- Understanding concepts (not specific pin mappings)
- Finding code examples and patterns

**Never** use web search for:
- XIAO-specific pin mappings
- Board-specific capabilities
- Platform support confirmation

### Verification Checklist

Before making changes to board specifications:

- [ ] Checked `Ref/PinOut/XIAO_Pinout_Reference.csv`
- [ ] Checked corresponding board wiki in `Ref/SeeedStudio_XIAO/`
- [ ] Cross-referenced information against both sources
- [ ] Confirmed information matches XIAO unified standard when applicable
- [ ] Flagged any conflicts or ambiguities for user review

### Example: RA4M1 I2C Verification

**Wrong approach**: Rely on CSV only → "RA4M1 has 1 I2C bus on D4/D5"

**Correct approach**:
1. CSV shows D4/D5 as default I2C
2. Wiki `Ref/SeeedStudio_XIAO/SeeedStudio_XIAO_RA4M1/XIAO-RA4M1.md` shows:
   - D4/D5 = I2C1
   - D6/D7 = I2C2 (alternate)
   - D15/D16 = I2C0 (alternate)
3. Conclusion: RA4M1 has 3 I2C buses total
