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

```python
auth = {
    'deveui': b'\x01\x02...\x08',
    'appeui': b'\x70\xB3...\x00',
    'appkey': b'\x2B\x7E...\xAE',
    'devnonce': b'\x00\x01'
}
lora.join(ActivationType.OTAA, auth)
lora.send_data("Hello OTAA")
```

---

## ⚙️ Configuration via JSON

Create a `lora_config.json` on your device to override default parameters:

```json
{
  "sf": 10,
  "output_power": 14,
  "invert_iq_rx": false
}
```

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