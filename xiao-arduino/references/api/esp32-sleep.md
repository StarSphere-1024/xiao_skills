# ESP32 Deep Sleep API

## Overview

ESP32 has multiple sleep modes (modem sleep / light sleep / deep sleep).

For XIAO ESP32 boards (ESP32-C3 / C5 / C6 / S3), **deep sleep** is the most common option for lowest power. When the chip wakes from deep sleep, it **boots from reset**. If you need to keep small state across deep sleep, store it in **RTC memory** using `RTC_DATA_ATTR`.

## Deep Sleep with Timer Wake

```cpp
#include "esp_sleep.h"

#define uS_TO_S_FACTOR 1000000ULL
#define TIME_TO_SLEEP  60

RTC_DATA_ATTR int bootCount = 0;

static void print_wakeup_reason() {
    esp_sleep_wakeup_cause_t wakeup_reason = esp_sleep_get_wakeup_cause();

    switch (wakeup_reason) {
        case ESP_SLEEP_WAKEUP_EXT0:     Serial.println("Wakeup caused by external signal using RTC_IO"); break;
        case ESP_SLEEP_WAKEUP_EXT1:     Serial.println("Wakeup caused by external signal using RTC_CNTL"); break;
        case ESP_SLEEP_WAKEUP_TIMER:    Serial.println("Wakeup caused by timer"); break;
        case ESP_SLEEP_WAKEUP_TOUCHPAD: Serial.println("Wakeup caused by touchpad"); break;
        case ESP_SLEEP_WAKEUP_ULP:      Serial.println("Wakeup caused by ULP program"); break;
        default:                        Serial.printf("Wakeup was not caused by deep sleep: %d\n", wakeup_reason); break;
    }
}

void setup() {
    Serial.begin(115200);
    delay(1000); // Give the Serial Monitor time

    ++bootCount;
    Serial.println("Boot number: " + String(bootCount));
    print_wakeup_reason();

    esp_sleep_enable_timer_wakeup(TIME_TO_SLEEP * uS_TO_S_FACTOR);
    Serial.println("Going to sleep now");
    Serial.flush();
    esp_deep_sleep_start();
}

void loop() {}
```

## RTC Memory (Preserves data through deep sleep)

```cpp
#include "esp_sleep.h"

RTC_DATA_ATTR int bootCount = 0;
RTC_DATA_ATTR float savedValue = 25.5f;

void setup() {
    Serial.begin(115200);
    delay(1000);

    ++bootCount;
    savedValue += 0.1f;

    Serial.printf("Boot count: %d\n", bootCount);
    Serial.printf("RTC savedValue: %.2f\n", savedValue);

    esp_sleep_enable_timer_wakeup(60 * 1000000ULL);
    Serial.flush();
    esp_deep_sleep_start();
}

void loop() {}
```

## External Wakeup (GPIO)

Different ESP32 variants support different wake APIs/pins.

:::note
On ESP32-C6, `ext0` wakeup is not supported, so the common approach is `ext1`.
:::

### Example (ESP32-C6: EXT1 wakeup)

```cpp
#include "esp_sleep.h"

// ESP32-C6 supports EXT1 wakeup on RTC-capable GPIOs (0-7 on C6)
#define BUTTON_PIN_BITMASK (1ULL << GPIO_NUM_0)

RTC_DATA_ATTR int bootCount = 0;

void setup() {
    Serial.begin(115200);
    delay(1000);

    ++bootCount;
    Serial.println("Boot number: " + String(bootCount));

    // Wake when GPIO0 goes HIGH
    esp_sleep_enable_ext1_wakeup(BUTTON_PIN_BITMASK, ESP_EXT1_WAKEUP_ANY_HIGH);

    Serial.println("Going to sleep now");
    Serial.flush();
    esp_deep_sleep_start();
}

void loop() {}
```

### Example (ESP32-C3 / ESP32-C5: GPIO wakeup)

```cpp
#include "esp_sleep.h"

// Example idea: wake on a GPIO level change.
// (Pin choices and levels depend on your board wiring.)

RTC_DATA_ATTR int bootCount = 0;

void setup() {
  Serial.begin(115200);
  delay(1000);

  ++bootCount;
  Serial.println("Boot number: " + String(bootCount));

  // Wake up when D1 goes LOW (example)
    uint64_t wakeup_mask = 1ULL << D1;
    esp_deep_sleep_enable_gpio_wakeup(wakeup_mask, ESP_GPIO_WAKEUP_GPIO_LOW);

  Serial.println("Going to sleep now");
  Serial.flush();
  esp_deep_sleep_start();
}

void loop() {}
```

## Light Sleep

```cpp
#include <WiFi.h>
#include "esp_wifi.h"

void setup() {
    Serial.begin(115200);
    // WiFi power-save mode (modem sleep / light sleep behavior depends on activity)
    esp_wifi_set_ps(WIFI_PS_MIN_MODEM);
}

void loop() {
    // Code continues to run
    // WiFi automatically sleeps between activity
    delay(1000);
}
```

## Notes

- After entering deep sleep, the USB serial port may disappear; you often need to **wake the board** to see the port again.
- For ESP32-C5, it is recommended to reserve JTAG pins and not use them as wake-up sources during low-power development.

## Best Practices

1. **Disable WiFi/BT**: Before sleep
2. **Use RTC memory**: For data preservation
3. **External wakeup**: Low power sensor monitoring
4. **Test current**: Use multimeter to measure
5. **Short wake time**: Minimize active time

## Troubleshooting

### Won't wake from deep sleep

1. Check wake source is enabled
2. Check GPIO pull-up/down
3. Verify timer value is correct

### WiFi won't reconnect after sleep

1. Always reinitialize WiFi after wake
2. Add delay before WiFi.begin()
3. Use WiFi.setAutoReconnect(true)

### High current in sleep

1. Ensure WiFi/BT are off
2. Check all pins set to INPUT_PULLDOWN
3. Verify no peripherals drawing power

## References

- `Ref/SeeedStudio_XIAO/SeeedStudio_XIAO_ESP32C3/XIAO_ESP32C3_Getting_Started.md`
- `Ref/SeeedStudio_XIAO/SeeedStudio_XIAO_ESP32C5/XIAO_ESP32C5_Getting_Started.md`
- `Ref/SeeedStudio_XIAO/SeeedStudio_XIAO_ESP32C6/XIAO_ESP32C6_Getting_Started.md`
- `Ref/SeeedStudio_XIAO/SeeedStudio_XIAO_ESP32S3/XIAO_ESP32S3_Sense_Consumption.md`
