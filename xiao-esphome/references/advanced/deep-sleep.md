# Deep Sleep Configuration

## Basic Deep Sleep

```yaml
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s  # 5 minutes
```

## Wake on Pin

```yaml
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s
  wakeup_pin: GPIO13
  wakeup_pin_mode: KEEP_AWAKE
```

## Wake on Touch

```yaml
deep_sleep:
  run_duration: 30s
  sleep_duration: 600s
  wakeup_pin: GPIO2
  wakeup_pin_mode: KEEP_AWAKE
```

## Wake on Timer Only

```yaml
deep_sleep:
  run_duration: 30s
  sleep_duration: 600s  # 10 minutes
```

## No Sleep Loop

```yaml
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s
  # Prevent entering sleep after boot
  # Useful for first-time setup
```

## ESP32 Deep Sleep

```yaml
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s
  wakeup_pin: GPIO13
  wakeup_pin_mode: KEEP_AWAKE

  # ESP32 specific options
  # wakeup_trigger:  # Optional
```

## RP2040 Deep Sleep

```yaml
# RP2040 deep sleep (restart on wake)
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s
```

## Pre-Sleep Actions

```yaml
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s

  # Run before sleeping
  on_enter_sleep:
    - logger.log: "Going to sleep..."

  # Run after waking
  on_wake:
    - logger.log: "Woke up!"
```

## Conditional Sleep

```yaml
# Only sleep if certain condition
deep_sleep:
  run_duration: 30s
  sleep_duration: 300s

script:
  - id: check_sleep
    mode: restart
    then:
      - if:
          condition:
            lambda: 'return id(battery_percent).state < 20;'
          then:
            - deep_sleep.enter: deep_sleep_1
```

## Battery Optimization

```yaml
deep_sleep:
  run_duration: 10s
  sleep_duration: 600s  # 10 minutes

wifi:
  fast_connect: true
  power_save_mode: high

logger:
  level: WARN  # Reduce logging
```

## Ultra Low Power

```yaml
deep_sleep:
  run_duration: 5s
  sleep_duration: 3600s  # 1 hour

# Disable everything
logger:
  level: NONE

# Fast connection
wifi:
  fast_connect: true
  power_save_mode: high

# Single sensor read
sensor:
  - platform: bme280
    update_interval: 5s  # Read once before sleeping
```

## Sleep Duration Based on Battery

```yaml
deep_sleep:
  run_duration: 30s

  # Variable sleep duration
  sleep_duration: !lambda "return id(battery_percent).state * 10s;"
```

## Wake Time

```yaml
# Specify when to wake
deep_sleep:
  run_duration: 30s
  # Sleep until specific time
  # Not supported on all platforms
```

## Troubleshooting

### Not Waking Up

1. Check wake pin configuration
2. Verify trigger mode
3. Check battery level
4. Look at boot logs

### Wakes Immediately

1. Check wake pin state
2. Verify run_duration
3. Check for button presses
4. Monitor debug logs

### Battery Drain

1. Increase sleep duration
2. Reduce run duration
3. Disable unnecessary components
4. Enable WiFi power save

## Best Practices

1. **Short run duration**: Minimize awake time
2. **Long sleep duration**: Extend battery life
3. **Use wake pins**: For event-driven operation
4. **Disable logging**: Reduce power
5. **Fast WiFi connect**: Use fast_connect
6. **Single sensor read**: Read once per cycle
7. **Monitor battery**: Adjust sleep based on level
