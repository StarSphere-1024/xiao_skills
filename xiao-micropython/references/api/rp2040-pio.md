# RP2040 MicroPython PIO (Programmable I/O)

## Overview

RP2040's PIO is a powerful co-processor that can implement custom protocols. Each PIO block has 4 state machines, and RP2040 has 2 PIO blocks.

## Basic PIO Example

```python
import machine
import rp2
from machine import Pin

@rp2.asm_pio(set=1, out=1)
def square_wave:
    pull()          [31]
    mov(x, osr)     [31]
    set(pins, 0)    [31]
    set(pins, 1)    [31]
    jmp(x_dec, square_wave) [31]

# Run PIO on pin 25
sm = rp2.StateMachine(0, square_wave, freq=2000, set_base=Pin(25))
sm.active(1)
```

## WS2812B (NeoPixel) PIO

```python
import rp2
from machine import Pin
import array

@rp2.asm_pio(sideset_init=rp2.PIO.OUT_LOW, out_shiftdir=rp2.PIO.SHIFT_LEFT,
            autopull=True, autopull_thresh=24)
def ws2812:
    out(null, 24)        .side(0) [5]
    mov(x, ~null)        .side(1) [3]
    label("loop")
    out(x, 1)            .side(0) [4]
    jmp(not_x, "end")    .side(1) [3]
    out(null, 24)        .side(0) [5]
    jmp("loop")           .side(1) [3]
    label("end")

class WS2812B:
    def __init__(self, pin, num_leds):
        self.sm = rp2.StateMachine(0, ws2812, freq=8000000,
                                  sideset_base=Pin(pin))
        self.sm.active(1)
        self.num_leds = num_leds
        self.ar = array.array("I", [0 for _ in range(num_leds)])

    def __setitem__(self, index, val):
        self.ar[index] = val

    def __getitem__(self, index):
        return self.ar[index]

    def show(self):
        for i in range(self.num_leds):
            self.sm.put(self.ar[i], 8)

# Use
leds = WS2812B(16, 8)  # GPIO16, 8 LEDs
leds[0] = (255, 0, 0)    # Red
leds.show()
```

## UART PIO Example

```python
@rp2.asm_pio(in_shiftdir=rp2.PIO.SHIFT_LEFT, autopull=True)
def uart_rx:
    pull(null)
    set(x, 7)             [7]
    label("loop")
    in_(pins, 1)
    jmp(x_dec, "loop")    [6]

# Configure UART
sm = rp2.StateMachine(0, uart_rx, in_base=Pin(0), jmp_pin=Pin(0))
sm.active(1)
```

## SPI PIO Example

```python
@rp2.asm_pio(sideset_init=rp2.PIO.OUT_LOW, out_shiftdir=rp2.PIO.SHIFT_LEFT,
            autopull=True, autopull_thresh=8)
def spi_csb:
    pull()                   [7]
    out(pindirs, 1)          [3]  # CS low
    set(pindirs, 1)          [4]  # CS high
    out(pins, 1)             [4]  # MOSI
    jmp(y_dec, "again")      [4]
    label("again")
```

## Custom Protocol Example

```python
# DHT11 protocol timing with PIO
@rp2.asm_pio(set_init=rp2.PIO.OUT_LOW, set_base=rp2.PIO.OUT_LOW)
def dht11:
    set(pins, 1)      [18000]  # Start signal (18ms high)
    set(pins, 0)      [30]     # Pull low
    wait(1, pin, 0)   [20]     # Wait for response
    set(pindirs, 1)    [20]     # Switch to input
    in_(pins, 1)       [30]     # Read data bit
    jmp(x_dec, "read_bit")
```

## PIO Assembly Reference

### Instructions

| Instruction | Description |
|-------------|-------------|
| `mov(dest, src)` | Move value |
| `set(pins, value)` | Set pins high/low |
| `out(pins, 1)` | Output from OSR to pins |
| `in_(pins, 1)` | Input from pins to ISR |
| `push()` | Push ISR to TX FIFO |
| `pull()` | Pull from RX FIFO to OSR |
| `jmp(label)` | Jump to label |
| `wait(pin, value)` | Wait for pin state |
| `irq(block)` | Set IRQ flag |

### Delay Cycles

```python
# Add delay after instruction
set(pins, 1)    [5]    # 5 cycle delay

# Side-set (delay while setting pin)
set(pins, 1)    .side(0) [3]  # Set pin, then 3 cycle delay
```

## State Machine Configuration

```python
sm = rp2.StateMachine(
    0,              # State machine number (0-3 per PIO)
    program,        # PIO assembly function
    freq=2000,      # Clock frequency
    set_base=Pin(0),# Base pin for set instruction
    in_base=Pin(1), # Base pin for in instruction
    out_base=Pin(2),# Base pin for out instruction
    sideset_base=Pin(3),  # Base pin for sideset
    jmp_pin=Pin(4)  # Pin for jmp condition
)
```

## Best Practices

1. **Start simple**: Test basic PIO first
2. **Calculate delays**: Use cycle counts for timing
3. **Use existing code**: Many PIO examples available
4. **Debug with logic analyzer**: Verify timing
5. **Check pin conflicts**: PIO pins exclusive
