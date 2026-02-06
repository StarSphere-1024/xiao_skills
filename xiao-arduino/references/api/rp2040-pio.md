# RP2040 PIO API for Arduino

## Overview

The Programmable I/O (PIO) on RP2040 allows custom peripherals and protocols.

## Basic PIO Assembly

### Simple Blink (PIO)

```cpp
#include <RP2040.h>

// PIO program for blinking
const uint16_t blink_program[] = {
    0x6001, // 0: out    pins, 1
    0xa002, // 1: mov    x, ~2
    0x4001, // 2: in     pins, 1
    0x0243, // 3: jmp    x--, 3
};

class PIOBlink {
private:
    PIO pio;
    uint sm;
    uint offset;

public:
    PIOBlink(uint pin) {
        pio = pio0;
        sm = 0;
        offset = pio_add_program(pio, blink_program);

        pio_sm_config c = blink_program_get_default_config(offset);
        sm_config_set_set_pins(&c, pin, 1);
        sm_config_set_out_pins(&c, pin, 1);
        sm_config_set_in_pins(&c, pin);
        sm_config_set_set_count(&c, 1);
        sm_config_set_out_count(&c, 1);

        pio_sm_init(pio, sm, offset, &c);
        pio_sm_set_enabled(pio, sm, true);
    }

    void set(bool state) {
        pio_sm_put_blocking(pio, sm, state);
    }
};

PIOBlink blink(LED_BUILTIN);

void setup() {
    Serial.begin(115200);
    Serial.println("PIO Blink started");
}

void loop() {
    static bool state = false;
    state = !state;
    blink.set(state);
    delay(1000);
}
```

## NeoPixel (WS2812B) PIO

### WS2812 Driver

```cpp
#include <RP2040.h>

// WS2812B PIO program
const uint16_t ws2812_program[] = {
    0xa5e2, // 0: set x, 2 [7]
    0xffff, // 1: mov osr, x ; pull ifempty [7] -- line gets patched
    0xb843, // 2: mov y, 9
    0xb062, // 3: out x, 32
    0x0143, // 4: jmp !x, 4
    0x1543, // 5: set osr, 1 ; mov y, 5
    0x1044, // 6: mov osr, x ; jmp y--, 6
    0x0042, // 7: nop          ; nop          ; pull block
    0xb542, // 8: nop          ; mov y, ~5
    0x4001, // 9: in pins, 1   ; mov osr, x
    0x0042, // 10: nop         ; nop         ; mov x, osr
    0x00c2, // 11: nop         ; push        ; nop
    0x0042, // 12: nop         ; nop         ; nop
    0xb5e2, // 13: set x, 6    ; mov y, ~5
    0x1043, // 14: mov osr, x  ; jmp y--, 14
};

class WS2812 {
private:
    PIO pio;
    uint sm;
    uint offset;

public:
    WS2812(uint pin, uint num_leds) {
        pio = pio0;
        sm = 0;
        offset = pio_add_program(pio, ws2812_program);

        pio_sm_config c = ws2812_program_get_default_config(offset);
        sm_config_set_set_pins(&c, pin, 1);

        pio_gpio_init(pio, pin);
        pio_sm_set_consecutive_pindirs(pio, sm, pin, 1, true);

        sm_config_set_out_shift(&c, false, true, 32);
        sm_config_set_fifo_join(&c, PIO_FIFO_JOIN_TX);
        sm_config_set_clkdiv_int_frac(&c, 4, 0);

        pio_sm_init(pio, sm, offset, &c);
        pio_sm_set_enabled(pio, sm, true);
    }

    void setPixel(uint8_t r, uint8_t g, uint8_t b) {
        uint32_t value = (g << 24) | (r << 16) | (b << 8);
        pio_sm_put_blocking(pio, sm, value);
    }

    void show() {
        // Push all pixels
    }
};

WS2812 strip(D6, 8);  // 8 LEDs on D6

void setup() {
    Serial.begin(115200);
    Serial.println("WS2812B initialized");
}

void loop() {
    static uint8_t hue = 0;

    // RGB cycle
    uint8_t r = hue;
    uint8_t g = 255 - hue;
    uint8_t b = hue >> 1;

    for (int i = 0; i < 8; i++) {
        strip.setPixel(r, g, b);
    }

    hue += 5;
    delay(50);
}
```

## UART PIO

### Software UART

```cpp
#include <RP2040.h>

// UART TX PIO program
const uint16_t uart_tx_program[] = {
    0x70a5, // 0: set pins, 1 [7]
    0x10c2, // 1: mov x, osr
    0x0043, // 2: jmp x--, 2
};

class PIOUART_TX {
private:
    PIO pio;
    uint sm;
    uint offset;

public:
    PIOUART_TX(uint tx_pin, uint baud) {
        pio = pio0;
        sm = 0;
        offset = pio_add_program(pio, uart_tx_program);

        pio_sm_config c = uart_tx_program_get_default_config(offset);
        sm_config_set_set_pins(&c, tx_pin, 1);

        // Set clock divider for baud rate
        float div = (float)clock_get_hz(clk_sys) / (8 * baud);
        sm_config_set_clkdiv(&c, div);

        pio_sm_init(pio, sm, offset, &c);
        pio_sm_set_enabled(pio, sm, true);
    }

    void write(uint8_t data) {
        pio_sm_put_blocking(pio, sm, (uint32_t)data << 24);
    }

    void print(const char* str) {
        while (*str) {
            write(*str++);
        }
    }
};

PIOUART_TX uart(D6, 9600);

void setup() {
    Serial.begin(115200);
    Serial.println("PIO UART TX initialized");
}

void loop() {
    uart.print("Hello from PIO!\n");
    delay(1000);
}
```

## SPI PIO

### SPI Master

```cpp
#include <RP2040.h>

// SPI Master program
const uint16_t spi_master_program[] = {
    0xa022, // 0: set x, 2
    0xa0a5, // 1: set pindirs, 1
    0x4001, // 2: in pins, 1
    0x07a3, // 3: mov osr, x ; nop ; jmp x--, 3
};

class PIOSPI {
private:
    PIO pio;
    uint sm;
    uint offset;

public:
    PIOSPI(uint sck, uint miso, uint mosi, uint cs, uint freq) {
        pio = pio0;
        sm = 0;
        offset = pio_add_program(pio, spi_master_program);

        pio_sm_config c = spi_master_program_get_default_config(offset);
        sm_config_set_in_pins(&c, miso);
        sm_config_set_set_pins(&c, sck);
        sm_config_set_out_pins(&c, mosi);

        // Set clock divider
        float div = (float)clock_get_hz(clk_sys) / freq;
        sm_config_set_clkdiv(&c, div);

        pio_sm_init(pio, sm, offset, &c);
        pio_sm_set_enabled(pio, sm, true);
    }

    uint8_t transfer(uint8_t data) {
        pio_sm_put_blocking(pio, sm, data);
        return pio_sm_get_blocking(pio, sm);
    }
};

PIOSPI spi(D7, D6, D5, D4, 1000000);

void setup() {
    Serial.begin(115200);
}

void loop() {
    uint8_t result = spi.transfer(0xAA);
    Serial.printf("SPI: 0x%02X\n", result);
    delay(1000);
}
```

## I2C PIO

### I2C Master

```cpp
#include <RP2040.h>

class PIOI2C {
private:
    PIO pio;
    uint sm;
    uint offset;
    uint sda, scl;

public:
    PIOI2C(uint sda_pin, uint scl_pin, uint baud) {
        sda = sda_pin;
        scl = scl_pin;
        pio = pio0;
        sm = 0;

        // I2C program would go here
        // This is a simplified example
    }

    void start() {
        // Generate START condition
    }

    void stop() {
        // Generate STOP condition
    }

    bool write(uint8_t data) {
        // Write byte and ACK
        return true;
    }

    uint8_t read(bool ack) {
        // Read byte
        return 0;
    }
};
```

## Rotary Encoder PIO

### Quadrature Decoder

```cpp
#include <RP2040.h>

// Quadrature encoder program
const uint16_t quad_program[] = {
    0x0083, // 0: jmp pin, 3
    0x0043, // 1: jmp x--, 1
    0xe026, // 2: set y, 2
    0x0000, // 3:
};

class PIOEncoder {
private:
    PIO pio;
    uint sm;
    uint offset;

public:
    PIOEncoder(uint pin_a, uint pin_b) {
        pio = pio0;
        sm = 0;
        offset = pio_add_program(pio, quad_program);

        pio_sm_config c = quad_program_get_default_config(offset);
        sm_config_set_in_pins(&c, pin_a);

        pio_sm_init(pio, sm, offset, &c);
        pio_sm_set_enabled(pio, sm, true);
    }

    int32_t getPosition() {
        // Read position from PIO
        return 0;
    }
};
```

## PWM PIO

### Custom PWM

```cpp
#include <RP2040.h>

class PIOPWM {
private:
    PIO pio;
    uint sm;
    uint offset;

public:
    PIOPWM(uint pin, uint freq, uint resolution) {
        pio = pio0;
        sm = 0;
        offset = pio_add_program(pio, /* PWM program */);

        pio_sm_config c = /* PWM config */;

        pio_sm_init(pio, sm, offset, &c);
        pio_sm_set_enabled(pio, sm, true);
    }

    void setDuty(uint16_t duty) {
        // Set duty cycle
    }
};
```

## Debugging PIO

### PIO Debugger

```cpp
void printPIOStatus() {
    Serial.println("=== PIO Status ===");
    Serial.printf("FIFO level: %d\n", pio0->sm[0].flevel);
    Serial.printf("Instruction: 0x%04X\n", pio0->sm[0].instr);
    Serial.printf("PC: %d\n", pio0->sm[0].pc);
    Serial.printf("X: %d\n", pio0->sm[0].x);
    Serial.printf("Y: %d\n", pio0->sm[0].y);
}

void setup() {
    Serial.begin(115200);
    printPIOStatus();
}

void loop() {}
```

## PIO Assembly Reference

### Instructions

| Instruction | Description |
|-------------|-------------|
| `mov dst, src` | Move value |
| `jmp cond, target` | Conditional jump |
| `wait x, y` | Wait for pin/status |
| `in src, count` | Read pins |
| `out dst, count` | Write pins |
| `push` | Push to RX FIFO |
| `pull` | Pull from TX FIFO |
| `set pins, value` | Set pins |
| `set pindirs, value` | Set pin directions |

### Conditions

| Condition | Description |
|-----------|-------------|
| `x--` | Decrement X |
| `y--` | Decrement Y |
| `x!=y` | X not equal Y |
| `pin` | Pin state |
| `!osre` | TX FIFO not empty |

## Tips

1. **State Machine Usage**: Use both SM0 and SM1 for parallel operations
2. **Clock Dividing**: Use `sm_config_set_clkdiv()` for timing
3. **FIFO Join**: Join TX/RX FIFO for single-direction transfers
4. **Interrupts**: Use PIO interrupts for events
5. **DMA**: Combine PIO with DMA for memory transfers
