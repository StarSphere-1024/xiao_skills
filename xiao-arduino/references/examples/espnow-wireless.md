# ESP-NOW Wireless Communication for XIAO ESP32

## Overview

ESP-NOW is a wireless communication protocol defined by Espressif that enables direct, fast, and low-power control of smart devices without the need for a router. It can coexist with Wi-Fi and Bluetooth LE.

## Features

- **Low Power**: Suitable for battery-powered devices
- **Fast Connection**: No complex pairing process required
- **Point-to-Point**: Direct communication between devices
- **Multiple Topologies**: One-to-one, one-to-many, many-to-one, many-to-many
- **Data Link Layer**: Simplified protocol stack for lower latency

## Board Support

| Board | ESP-NOW Support |
|-------|----------------|
| ESP32C3 | ✅ |
| ESP32C5 | ✅ |
| ESP32C6 | ✅ |
| ESP32S3 | ✅ |
| nRF52840 | ❌ (use BLE) |
| RP2040 | ❌ (no WiFi) |
| SAMD21 | ❌ (no WiFi) |
| RA4M1 | ❌ (no WiFi) |

## Comparison with Other Protocols

| Protocol | Power | Range | Pairing | Use Case |
|----------|-------|-------|---------|----------|
| **ESP-NOW** | Low | Medium | No pairing | Sensor networks, remote control |
| **Wi-Fi** | High | Long | SSID/Password | Internet connectivity |
| **BLE** | Medium | Short | Pairing required | Wearables, mobile apps |

## Basic Sender/Receiver Example

### Step 1: Get MAC Address

First, upload this code to each XIAO to get its MAC address:

```cpp
#include <WiFi.h>

void setup() {
    Serial.begin(115200);
    WiFi.mode(WIFI_STA);
    WiFi.STA.begin();

    while (!WiFi.STA.started()) {
        delay(100);
    }

    uint8_t mac[6];
    WiFi.macAddress(mac);

    Serial.print("MAC Address: ");
    Serial.printf("0x%02x, 0x%02x, 0x%02x, 0x%02x, 0x%02x, 0x%02x",
                 mac[0], mac[1], mac[2], mac[3], mac[4], mac[5]);
    Serial.println();
}

void loop() {}
```

### Step 2: Sender Code

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>

#define ESPNOW_WIFI_CHANNEL 0
#define BAUD 115200

// Replace with your receiver's MAC address
static uint8_t receiver_mac[6] = {0xYY, 0xYY, 0xYY, 0xYY, 0xYY, 0xYY};

// Data structure
typedef struct struct_message {
    char device[20];
    char data[32];
} struct_message;

struct_message outgoingData;
struct_message incomingData;

// Callback when data is sent
void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
    Serial.print("Send status: ");
    Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Success" : "Fail");
}

// Callback when data is received
void OnDataRecv(const esp_now_recv_info *info, const uint8_t *incomingData, int len) {
    memcpy(&incomingData, incomingData, sizeof(incomingData));
    Serial.print("Received from ");
    Serial.print(incomingData.device);
    Serial.print(": ");
    Serial.println(incomingData.data);
}

void setup() {
    Serial.begin(BAUD);

    // Set WiFi mode
    WiFi.mode(WIFI_STA);

    // Initialize ESP-NOW
    if (esp_now_init() != ESP_OK) {
        Serial.println("Error initializing ESP-NOW");
        return;
    }

    // Register callbacks
    esp_now_register_send_cb(OnDataSent);
    esp_now_register_recv_cb(OnDataRecv);

    // Register peer
    esp_now_peer_info_t peerInfo;
    memcpy(peerInfo.peer_addr, receiver_mac, 6);
    peerInfo.channel = ESPNOW_WIFI_CHANNEL;
    peerInfo.encrypt = false;

    if (esp_now_add_peer(&peerInfo) != ESP_OK) {
        Serial.println("Failed to add peer");
    } else {
        Serial.println("Peer added successfully");
    }
}

void loop() {
    // Prepare data
    strcpy(outgoingData.device, "XIAO-Sender");
    strcpy(outgoingData.data, "Hello Receiver!");

    // Send message
    esp_err_t result = esp_now_send(receiver_mac, (uint8_t *)&outgoingData, sizeof(outgoingData));

    if (result == ESP_OK) {
        Serial.println("Sent with success");
    } else {
        Serial.println("Error sending the data");
    }

    delay(2000);
}
```

### Step 3: Receiver Code

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>

#define ESPNOW_WIFI_CHANNEL 0
#define BAUD 115200

// Replace with your sender's MAC address
static uint8_t sender_mac[6] = {0xXX, 0xXX, 0xXX, 0xXX, 0xXX, 0xXX};

typedef struct struct_message {
    char device[20];
    char data[32];
} struct_message;

struct_message outgoingData;
struct_message incomingData;

void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
    Serial.print("Send status: ");
    Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Success" : "Fail");
}

void OnDataRecv(const esp_now_recv_info *info, const uint8_t *incomingData, int len) {
    memcpy(&incomingData, incomingData, sizeof(incomingData));

    Serial.print("Received from ");
    Serial.print(incomingData.device);
    Serial.print(": ");
    Serial.println(incomingData.data);

    // Send response
    strcpy(outgoingData.device, "XIAO-Receiver");
    strcpy(outgoingData.data, "Got it!");
    esp_now_send(info->src_addr, (uint8_t *)&outgoingData, sizeof(outgoingData));
}

void setup() {
    Serial.begin(BAUD);
    WiFi.mode(WIFI_STA);

    if (esp_now_init() != ESP_OK) {
        Serial.println("Error initializing ESP-NOW");
        return;
    }

    esp_now_register_send_cb(OnDataSent);
    esp_now_register_recv_cb(OnDataRecv);

    esp_now_peer_info_t peerInfo;
    memcpy(peerInfo.peer_addr, sender_mac, 6);
    peerInfo.channel = ESPNOW_WIFI_CHANNEL;
    peerInfo.encrypt = false;

    if (esp_now_add_peer(&peerInfo) != ESP_OK) {
        Serial.println("Failed to add peer");
    } else {
        Serial.println("Peer added successfully");
    }
}

void loop() {}
```

## Sensor Network Example (One Sender, Multiple Receivers)

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>

// Multiple receivers
static uint8_t receiver1_mac[6] = {0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF};
static uint8_t receiver2_mac[6] = {0x11, 0x22, 0x33, 0x44, 0x55, 0x66};
static uint8_t receiver3_mac[6] = {0x77, 0x88, 0x99, 0xAA, 0xBB, 0xCC};

typedef struct struct_sensor {
    int sensorId;
    float temperature;
    float humidity;
    unsigned long timestamp;
} struct_sensor;

struct_sensor sensorData;

void setup() {
    Serial.begin(115200);
    WiFi.mode(WIFI_STA);

    if (esp_now_init() != ESP_OK) {
        Serial.println("Error initializing ESP-NOW");
        return;
    }

    // Register all peers
    esp_now_peer_info_t peerInfo;
    peerInfo.channel = 0;
    peerInfo.encrypt = false;

    uint8_t* peers[] = {receiver1_mac, receiver2_mac, receiver3_mac};

    for (int i = 0; i < 3; i++) {
        memcpy(peerInfo.peer_addr, peers[i], 6);
        if (esp_now_add_peer(&peerInfo) != ESP_OK) {
            Serial.printf("Failed to add peer %d\n", i + 1);
        }
    }

    Serial.println("ESP-NOW Multi-cast Ready");
}

void loop() {
    // Simulate sensor readings
    sensorData.sensorId = 1;
    sensorData.temperature = random(200, 300) / 10.0;
    sensorData.humidity = random(400, 800) / 10.0;
    sensorData.timestamp = millis();

    // Broadcast to all receivers
    esp_now_send(NULL, (uint8_t *)&sensorData, sizeof(sensorData));

    Serial.printf("Sent: T=%.1f°C, H=%.1f%%\n",
                 sensorData.temperature, sensorData.humidity);

    delay(5000);
}
```

## Bidirectional Communication

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>

static uint8_t peer_mac[6] = {0xYY, 0xYY, 0xYY, 0xYY, 0xYY, 0xYY};

typedef struct struct_message {
    char device[20];
    int counter;
    float value;
} struct_message;

struct_message outgoingData;
struct_message incomingData;

int messageCount = 0;

void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
    Serial.print("Last packet sent to: ");
    Serial.printf("%02x:%02x:%02x:%02x:%02x:%02x",
                 mac_addr[0], mac_addr[1], mac_addr[2],
                 mac_addr[3], mac_addr[4], mac_addr[5]);
    Serial.print(", status: ");
    Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Success" : "Fail");
}

void OnDataRecv(const esp_now_recv_info *info, const uint8_t *incomingData, int len) {
    memcpy(&incomingData, incomingData, sizeof(incomingData));

    Serial.print("Received from ");
    Serial.print(incomingData.device);
    Serial.print(": counter=");
    Serial.print(incomingData.counter);
    Serial.print(", value=");
    Serial.println(incomingData.value);

    // Send acknowledgment
    messageCount++;
    strcpy(outgoingData.device, WiFi.macAddress().c_str());
    outgoingData.counter = messageCount;
    outgoingData.value = incomingData.value * 1.1;

    esp_now_send(info->src_addr, (uint8_t *)&outgoingData, sizeof(outgoingData));
}

void setup() {
    Serial.begin(115200);
    WiFi.mode(WIFI_STA);

    if (esp_now_init() != ESP_OK) {
        Serial.println("Error initializing ESP-NOW");
        return;
    }

    esp_now_register_send_cb(OnDataSent);
    esp_now_register_recv_cb(OnDataRecv);

    // Set device name as MAC
    String mac = WiFi.macAddress();
    strcpy(outgoingData.device, mac.c_str());

    esp_now_peer_info_t peerInfo;
    memcpy(peerInfo.peer_addr, peer_mac, 6);
    peerInfo.channel = 0;
    peerInfo.encrypt = false;

    if (esp_now_add_peer(&peerInfo) == ESP_OK) {
        Serial.println("Peer added successfully");
    }
}

void loop() {
    // Send periodic heartbeat
    messageCount++;
    outgoingData.counter = messageCount;
    outgoingData.value = random(1000) / 100.0;

    esp_now_send(peer_mac, (uint8_t *)&outgoingData, sizeof(outgoingData));
    delay(5000);
}
```

## Encrypted Communication

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>

static uint8_t peer_mac[6] = {0xYY, 0xYY, 0xYY, 0xYY, 0xYY, 0xYY};

// Primary Master Key (must match on both devices)
static const char* PMK_KEY = "ThisIsASecretKey_16";  // Exactly 16 bytes

// Local Master Key (must match on both devices)
static const char* LMK_KEY = "AnotherSecretKey16";  // Exactly 16 bytes

void setup() {
    Serial.begin(115200);
    WiFi.mode(WIFI_STA);

    if (esp_now_init() != ESP_OK) {
        Serial.println("Error initializing ESP-NOW");
        return;
    }

    // Set encryption keys
    esp_now_set_pmk((uint8_t *)PMK_KEY);

    esp_now_peer_info_t peerInfo;
    memcpy(peerInfo.peer_addr, peer_mac, 6);
    peerInfo.channel = 0;
    peerInfo.encrypt = true;

    // Set LMK for this peer
    memcpy(peerInfo.lmk, LMK_KEY, 16);

    if (esp_now_add_peer(&peerInfo) == ESP_OK) {
        Serial.println("Encrypted peer added");
    }
}

void loop() {
    // Send encrypted messages
    const char* message = "Secret Data";
    esp_now_send(peer_mac, (uint8_t *)message, strlen(message) + 1);
    delay(2000);
}
```

## ESP-NOW with Deep Sleep

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>
#include <esp_sleep.h>

static uint8_t receiver_mac[6] = {0xYY, 0xYY, 0xYY, 0xYY, 0xYY, 0xYY};

typedef struct struct_sensor {
    float temperature;
    float battery;
    unsigned long uptime;
} struct_sensor;

struct_sensor sensorData;

void setup() {
    Serial.begin(115200);

    // Initialize WiFi station mode
    WiFi.mode(WIFI_STA);

    // Initialize ESP-NOW
    if (esp_now_init() != ESP_OK) {
        Serial.println("ESP-NOW init failed");
        gotoSleep();
        return;
    }

    // Add peer
    esp_now_peer_info_t peerInfo;
    memcpy(peerInfo.peer_addr, receiver_mac, 6);
    peerInfo.channel = 0;
    peerInfo.encrypt = false;

    if (esp_now_add_peer(&peerInfo) != ESP_OK) {
        Serial.println("Failed to add peer");
        gotoSleep();
        return;
    }

    // Read sensors (simulated)
    sensorData.temperature = random(150, 250) / 10.0;
    sensorData.battery = (float)analogRead(A0) / 4095.0 * 3.3 * 2;
    sensorData.uptime = millis();

    // Send data
    esp_err_t result = esp_now_send(receiver_mac, (uint8_t *)&sensorData, sizeof(sensorData));

    if (result == ESP_OK) {
        Serial.println("Data sent successfully");
    } else {
        Serial.println("Error sending data");
    }

    // Go to sleep for 60 seconds
    gotoSleep();
}

void gotoSleep() {
    Serial.println("Going to sleep...");
    delay(100);
    esp_sleep_enable_timer_wakeup(60 * 1000000);  // 60 seconds
    esp_deep_sleep_start();
}

void loop() {}
```

## Multiple ESP-NOW Networks (Channel Selection)

```cpp
#include <Arduino.h>
#include <WiFi.h>
#include <esp_now.h>

// Use different channels for separate networks
#define ESPNOW_NETWORK_1_CHANNEL 1
#define ESPNOW_NETWORK_2_CHANNEL 6

// Network 1 peers
static uint8_t network1_peer[6] = {0xAA, 0xAA, 0xAA, 0xAA, 0xAA, 0xAA};

// Network 2 peers
static uint8_t network2_peer[6] = {0xBB, 0xBB, 0xBB, 0xBB, 0xBB, 0xBB};

void setup() {
    Serial.begin(115200);

    // Set WiFi channel before ESP-NOW init
    WiFi.mode(WIFI_STA);
    WiFi.setChannel(ESPNOW_NETWORK_1_CHANNEL);

    if (esp_now_init() != ESP_OK) {
        Serial.println("ESP-NOW init failed");
        return;
    }

    // Add peer for network 1
    esp_now_peer_info_t peerInfo1;
    memcpy(peerInfo1.peer_addr, network1_peer, 6);
    peerInfo1.channel = ESPNOW_NETWORK_1_CHANNEL;
    peerInfo1.encrypt = false;
    esp_now_add_peer(&peerInfo1);

    Serial.printf("ESP-NOW ready on channel %d\n", ESPNOW_NETWORK_1_CHANNEL);
}

void loop() {
    const char* msg = "Message for Network 1";
    esp_now_send(network1_peer, (uint8_t *)msg, strlen(msg) + 1);
    delay(3000);
}
```

## ESP-NOW API Reference

### Initialization

| Function | Description |
|----------|-------------|
| `esp_now_init()` | Initialize ESP-NOW |
| `esp_now_deinit()` | Deinitialize ESP-NOW |
| `esp_now_get_version()` | Get ESP-NOW version |

### Peer Management

| Function | Description |
|----------|-------------|
| `esp_now_add_peer()` | Add a peer device |
| `esp_now_remove_peer()` | Remove a peer device |
| `esp_now_del_peer()` | Delete all peers |
| `esp_now_mod_peer()` | Modify peer info |
| `esp_now_get_peer()` | Get peer info |
| `esp_now_fetch_peer()` | Fetch peer (for iteration) |

### Data Transfer

| Function | Description |
|----------|-------------|
| `esp_now_send()` | Send data to peer |
| `esp_now_register_send_cb()` | Register send callback |
| `esp_now_register_recv_cb()` | Register receive callback |

### Encryption

| Function | Description |
|----------|-------------|
| `esp_now_set_pmk()` | Set Primary Master Key |
| `esp_now_get_pmk()` | Get Primary Master Key |

## Troubleshooting

### No Communication

1. **Check MAC addresses**: Ensure MAC addresses are correct
2. **Same channel**: All devices must be on same Wi-Fi channel
3. **Reset devices**: Power cycle both sender and receiver
4. **Check peer registration**: Verify `esp_now_add_peer()` returns ESP_OK

### Incomplete Data Reception

1. **Structure size mismatch**: Ensure sender/receiver have same struct sizes
2. **Buffer overflow**: Keep messages under 250 bytes
3. **Memory alignment**: Use `__attribute__((packed))` if needed

### High Power Consumption

1. **Reduce transmission frequency**: Increase delay between sends
2. **Use deep sleep**: Put device to sleep between transmissions
3. **Lower transmit power**: Use `WiFi.setTxPower()`

### Range Issues

1. **Antenna orientation**: Position antennas optimally
2. **Interference**: Change Wi-Fi channel
3. **Obstacles**: Clear line of sight works best

## Best Practices

1. **Always check return values**: Verify ESP_OK status
2. **Use fixed structure sizes**: Keep data structures consistent
3. **Implement retransmission**: Add retry logic for critical messages
4. **Add sequence numbers**: Detect missing packets
5. **Use encryption for security**: Enable encryption for sensitive data
6. **Test range in actual environment**: Real-world conditions vary

## Resources

- [ESP32 ESP-NOW Guide](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/network/esp_now.html)
- [ESP-NOW Wikipedia](https://en.wikipedia.org/wiki/ESP-NOW)
