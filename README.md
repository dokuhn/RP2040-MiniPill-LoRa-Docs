# 📡 PyLoRaWAN

[![MicroPython](https://img.shields.io/badge/platform-MicroPython-blue.svg)](https://micropython.org/)
[![LoRaWAN](https://img.shields.io/badge/protocol-LoRaWAN-orange.svg)](https://lora-alliance.org/)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

A lightweight, MicroPython-based LoRaWAN communication stack for SX1276 LoRa modules.  
Supports **ABP** and **OTAA** activation, channel hopping, MAC command handling, and configurable region-specific settings (EU868, US915).

---

## ✨ Features

- ✅ ABP and OTAA activation support
- 🌍 Regional frequency plans: EU868, US915
- 🔄 Automatic channel switching
- 📦 JSON-based config override (`lora_config.json`)
- ⚙️ Full MAC command processing (e.g. LinkADRReq, DevStatusReq, etc.)
- 💾 Frame counter persistence (`fcnt.json`)
- 🔊 RX1/RX2 receive window logic

---

## 📁 File Structure

| File              | Description                                      |
|-------------------|--------------------------------------------------|
| `py_lorawan.py`   | Main class handling LoRaWAN protocol             |
| `lora_config.json`| Optional file to override default LoRa settings |
| `fcnt.json`       | Frame counter persistence between reboots        |

---

## 🛠 Installation

### Requirements

- RP2040-MiniPill-LoRa board

---

## 🚀 Quick Start

### ABP Activation

```python
from py_lorawan import PyLoRaWAN, ActivationType

lora = PyLoRaWAN(spi, cs)
auth = {
    'devaddr': '26011F33',
    'nwskey': [0x00, 0x01, ...],  # 16-byte list
    'appskey': [0x00, 0x01, ...]
}
lora.join(ActivationType.ABP, auth)
lora.send_test_msg()
```


### OTAA Activation

This example demonstrates how to use PyLoRaWAN with **Over-The-Air Activation (OTAA)** on the RP2040-MiniPill-LoRa board. The device will join a LoRaWAN network and send a test message repeatedly.

```python
from machine import Pin, SPI
import machine
import time
from random import randrange

from PyLoRaWAN import PyLoRaWAN, ActivationType

# LoRaWAN OTAA credentials (replace with your actual values)
deveui = [0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]
appeui = [0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00]
appkey = [0x00] * 16
devnonce = [randrange(256), randrange(256)]

# Setup SPI and Chip Select for the LoRa module
spi = machine.SPI(0,
    baudrate=100_000,
    polarity=1,
    phase=1,
    bits=8,
    firstbit=machine.SPI.MSB,
    sck=machine.Pin('GP2'),
    mosi=machine.Pin('GP3'),
    miso=machine.Pin('GP4'))

cs = machine.Pin('GP5')

# Optional startup delay
time.sleep(10)

# Initialize LoRaWAN and join using OTAA
lorawan = PyLoRaWAN(spi, cs)
lorawan.join(
    ActivationType.OTAA,
    auth={
        'deveui': deveui,
        'appeui': appeui,
        'appkey': appkey,
        'devnonce': devnonce
    }
)

# Send test message in loop
if __name__ == '__main__':
    try:
        while True:
            lorawan.send_test_msg()
            time.sleep(30)
    except KeyboardInterrupt:
        print("\nProgram terminated by user.")
```

---

## ⚙️ Overriding Default LoRa Parameters via `lora_config.json`

The `PyLoRaWAN` library supports dynamic overriding of the default LoRa modem parameters using a configuration file named `lora_config.json`. This file can be placed directly on the microcontroller's file system and allows for quick parameter adjustments without changing the main source code.

### 🧐 Why Use It

- **Platform Flexibility**: Deploy the same script on multiple devices by simply modifying the config file.
- **Runtime Customization**: Easily adapt frequencies, bandwidth, power, and more.
- **No Reflashing Needed**: Make changes without updating firmware (for e.g. with the smartphone). 

### 📁 Filename & Location

The file must be saved at the root of the device's filesystem as `lora_config.json`.

### 📄 Example: `lora_config.json`

```json
{
  "freq_khz": 868500,
  "sf": 10,
  "coding_rate": 5,
  "bw": "125",
  "tx_ant": "PA_BOOST",
  "output_power": 14,
  "syncword": 52,
  "preamble_len": 8,
  "invert_iq_rx": true,
  "crc_en": true
}
```

### 🧱 Parameter Descriptions

| Key             | Description |
|----------------|-------------|
| `freq_khz`     | Operating frequency in kilohertz (e.g., 868100 for 868.1 MHz) |
| `sf`           | Spreading Factor (valid range: 7–12) |
| `coding_rate`  | FEC coding rate (e.g., 5 for 4/5) |
| `bw`           | Bandwidth in kHz (`"125"`, `"250"`, or `"500"`) |
| `tx_ant`       | TX antenna path (e.g., `"PA_BOOST"`, `"RFO"`) |
| `output_power` | Output power in dBm |
| `syncword`     | Sync word (e.g., `0x34` for LoRaWAN) |
| `preamble_len` | Preamble length (default is 8) |
| `invert_iq_rx` | Invert IQ on RX (true for LoRaWAN) |
| `crc_en`       | Enable CRC for payload (boolean) |

### 💡 How It Works

During initialization, the library attempts to read and apply settings from `lora_config.json`:

```python
self._load_default_lora_cfg()
```

If the file exists, each recognized field will update the internal configuration (`default_lora_cfg`). Unknown fields will be ignored with a log message.

**Core logic in `_load_default_lora_cfg()`:**

```python
if CONFIG_FILE in os.listdir():
    with open(CONFIG_FILE, "r") as f:
        config = ujson.load(f)
    for key, value in config.items():
        if key in self.default_lora_cfg:
            self.default_lora_cfg[key] = value
```

### 🔍 Runtime Feedback

On boot, the library provides console output about:

- Which parameters were applied from file
- Any unrecognized entries
- If the file is missing or malformed

### 🔧 Best Practices

- Prepare per-device `lora_config.json` files to customize behavior for different deployments.
- Use consistent field names and valid values to avoid silent fallbacks.
- Include a comment block in your firmware reminding users of this feature.

---

This configuration flexibility makes it easy to adapt your firmware to regional regulations or test environments without recompilation or re-deployment.

---

## 🧩 Class Overview: `PyLoRaWAN`

### Initialization

```python
PyLoRaWAN(spi, cs, region=Region.EU868)
```

- `spi`: MicroPython SPI instance
- `cs`: Chip Select pin
- `region`: Frequency plan dict (default: `EU868`)

---

### Key Methods

| Method | Description |
|--------|-------------|
| `join(activation, auth)` | Connect via ABP or OTAA |
| `send_data(data, timeout=8000)` | Send a string uplink |
| `send_test_msg()` | Sends `"Hallo"` for testing |
| `select_next_channel()` | Cycle to the next frequency |
| `_handle_rx_windows()` | Manages RX1 and RX2 receive logic |
| `_process_downlink(payload)` | Handles MAC command responses |

---

### MAC Command Handling

Built-in support for parsing and responding to:
- `LinkCheckReq`
- `LinkADRReq`
- `DutyCycleReq`
- `RXParamSetupReq`
- `DevStatusReq`
- `NewChannelReq`
- `RXTimingSetupReq`

---

## 🧠 Regional Support

### `Region.EU868`
```python
{
  "freq_khz": [868100, 868300, 868500],
  "sf": [7, 8, 9, 10, 11, 12],
  "rx2_freq_khz": 869525,
  "rx2_sf": 12,
  ...
}
```

### `Region.US915`
```python
{
  "freq_khz": [902300, 902500, 902700],
  "sf": [7, 8, 9, 10, 11, 12],
  "rx2_freq_khz": 923300,
  "rx2_sf": 12,
  ...
}
```

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).

---

## 🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you’d like to change.

---

## 📬 Contact

Created by **@dokuhn** – feel free to reach out via GitHub Issues or a PR.

---

## 🧪 TODOs

- [ ] Add confirmed uplink support
- [ ] Class for structured uplink/downlink payloads
- [ ] Retry logic for join requests

---