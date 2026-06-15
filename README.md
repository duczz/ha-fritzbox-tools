<div align="center">

# FRITZ!Box Tools

### A patched & improved FRITZ!Box Tools integration for Home Assistant

[![HACS Custom][hacs-badge]][hacs-url]
[![Home Assistant][ha-badge]][ha-url]
[![Version][version-badge]][release-url]
[![License][license-badge]](LICENSE)

[![Add to HACS][hacs-add-badge]][hacs-add-url]

</div>

---

This fork of the official Home Assistant [FRITZ!Box Tools integration][ha-url] fixes an upstream bug that caused the guest Wi-Fi QR image entity to trigger repeated `>10s update warnings` on slower devices.

## 🔌 Supported Platforms

| Platform | Description |
|---|---|
| 🚨 `binary_sensor` | Connection status, DSL link status |
| 🔘 `button` | Wake-on-LAN for connected devices |
| 📡 `device_tracker` | Track devices on the local network |
| 🖼️ `image` | FRITZ!Box QR code for guest Wi-Fi |
| 📊 `sensor` | IP addresses, uptime, throughput, DSL stats, signal quality |
| 🔀 `switch` | Wi-Fi networks, port forwardings, call deflections, access profiles |
| 🔄 `update` | FRITZ!OS firmware update notifications |

<details>
<summary>📋 Sensor details</summary>

| Sensor | Unit | Description |
|---|---|---|
| External IP | — | Public IPv4 address |
| External IPv6 | — | Public IPv6 address |
| Connection uptime | — | Time since last internet connection |
| Upload speed | kB/s | Current upstream throughput |
| Download speed | kB/s | Current downstream throughput |
| Max. upload speed | kB/s | DSL sync speed upstream |
| Max. download speed | kB/s | DSL sync speed downstream |
| Uploaded data | GB | Total data sent |
| Downloaded data | GB | Total data received |
| Link upload speed | kB/s | Physical layer upstream rate |
| Link download speed | kB/s | Physical layer downstream rate |
| Noise margin (up/down) | dB | DSL noise margin |
| Attenuation (up/down) | dB | DSL line attenuation |

</details>

---

## ✅ Requirements

- Home Assistant **2024.1.0** or newer
- A FRITZ!Box router (AVM)
- **UPnP** must be enabled in the FRITZ!Box UI (*Home Network → Network → General*)

---

## 🚀 Installation

### HACS (recommended)

1. Open **HACS** in Home Assistant
2. Go to **Integrations** → three-dot menu → **Custom repositories**
3. Add this repository URL, category: **Integration**
4. Search for **FRITZ!Box Tools** and install
5. Restart Home Assistant

### Manual

1. Copy all files into:
   ```
   <config>/custom_components/fritz/
   ```
2. Restart Home Assistant

---

## ⚙️ Configuration

**Settings → Devices & Services → Add Integration → FRITZ!Box Tools**

| Field | Description |
|---|---|
| Host | Hostname or IP (default: `192.168.178.1`) |
| Port | UPnP port (default: `49000`, SSL: `49443`) |
| Username | FRITZ!Box user account |
| Password | Password for the user account |
| SSL | Enable encrypted connection (optional) |

Auto-discovery via SSDP is supported — if your FRITZ!Box is on the local network, it may appear automatically.

> **Tip:** Create a dedicated FRITZ!Box user with the required permissions instead of using the admin account.

---

## 🛠️ Improvements over the official integration

### 🐛 Bug Fixes

- **Guest Wi-Fi QR image no longer causes `>10s` update warnings** — The QR image entity used `_attr_should_poll = True` and performed an independent blocking HTTP request to the FRITZ!Box on every HA polling cycle, on top of all coordinator requests. On slower devices (e.g. FRITZ!Box 7490) this triggered the `Update of image.fritz_box_* is taking over 10 seconds` warning multiple times per day ([#159545](https://github.com/home-assistant/core/issues/159545)).

  The fix moves the QR code fetch into the existing coordinator (`_async_update_data`), rewrites `FritzGuestWifiQRImage` to extend `CoordinatorEntity` with `_attr_should_poll = False`, and reacts to updates via `_handle_coordinator_update` — no extra HTTP request. ([`coordinator.py`](homeassistant/components/fritz/coordinator.py), [`image.py`](homeassistant/components/fritz/image.py))

- **No ERROR log on connection loss during service calls** — When a `ConnectionError` occurs inside `_async_service_call` (e.g. during nightly DSL forced reconnects), the integration now silently schedules a reload instead of writing an ERROR entry to the HA log. Consistent with the existing behavior in `_async_update_data`. (`coordinator.py`)

- **Retry on RemoteDisconnected (`ConnectionError`)** — Stale pooled HTTP keep-alive connections that are silently closed by the FRITZ!Box now trigger a single transparent retry instead of crashing the update loop and reloading the integration.

### ✨ New Features

- **More detailed diagnostics** — Diagnostics export now includes device uptime and extra firmware information to help with troubleshooting (backported from upstream PR #3).

---

## 🤝 Contributing

Pull requests are welcome. For larger changes, please open an issue first to discuss your approach.

---

[hacs-badge]: https://img.shields.io/badge/HACS-Custom-orange.svg?style=for-the-badge&logo=homeassistantcommunitystore&logoColor=white
[hacs-url]: https://hacs.xyz
[ha-badge]: https://img.shields.io/badge/Home%20Assistant-2024.1+-41BDF5.svg?style=for-the-badge&logo=homeassistant&logoColor=white
[ha-url]: https://www.home-assistant.io/integrations/fritz
[version-badge]: https://img.shields.io/badge/version-1.1.0-22c55e.svg?style=for-the-badge&logo=github&logoColor=white
[release-url]: https://github.com/duczz/fritzbox-tools/releases
[license-badge]: https://img.shields.io/badge/license-MIT-94a3b8.svg?style=for-the-badge
[hacs-add-badge]: https://my.home-assistant.io/badges/hacs_repository.svg
[hacs-add-url]: https://my.home-assistant.io/redirect/hacs_repository/?owner=duczz&repository=fritzbox-tools&category=integration
