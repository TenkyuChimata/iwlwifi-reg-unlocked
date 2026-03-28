# linux-wireless-reg-unlocked

Patched Intel **iwlwifi** and Linux **cfg80211/regulatory** subsystems to override firmware and regulatory restrictions, exposing additional Wi-Fi capabilities (including 6 GHz).

This project provides external kernel modules for Arch Linux that modify both the Intel iwlwifi driver and the cfg80211 regulatory layer, allowing advanced users to bypass multiple layers of wireless restriction without rebuilding the full kernel.

---

## 🚀 Features

### iwlwifi (driver / NVM layer)

- Add `lar_disable` module parameter
- Disable Intel iwlwifi LAR (Location Aware Regulatory)
- Override NVM channel flags:
  - Force-enable `NVM_CHANNEL_VALID`
  - Force-enable active transmission (`ACTIVE`, `IBSS`)
  - Remove `NO_IR` restrictions (allow AP / initiating transmissions)
- Expose channels normally filtered out by firmware/NVM
- Relax channel capability restrictions (bandwidth / usage)
- Force-enable additional 6 GHz channel capabilities (VLP / AFC related flags)

### cfg80211 (regulatory layer)

- Allow unsigned regulatory database (regdb)
- Disable regulatory intersection logic
- Ignore driver-provided regulatory hints
- Ignore Country IE regulatory updates
- Prevent cfg80211 from overriding driver-exposed capabilities

---

## ⚠️ Disclaimer

> 🚨 IMPORTANT

This project **intentionally bypasses regulatory enforcement mechanisms** at multiple layers:

- Firmware (NVM interpretation)
- Driver (iwlwifi)
- Kernel regulatory layer (cfg80211)

This may:

- Violate local wireless communication laws
- Cause interference with licensed spectrum users (e.g. DFS radar bands)
- Enable transmission where it is normally prohibited

This project is **for research and experimental purposes only**.

You are fully responsible for compliance with local regulations.

---

## 📥 Installation

### Manual build

```bash
git clone https://github.com/TenkyuChimata/linux-wireless-reg-unlocked.git
cd linux-wireless-reg-unlocked
makepkg -si
```

---

## 🔧 Usage

Reboot after installation to load the modified modules.

This package installs a modprobe configuration enabling:

```bash
options iwlwifi lar_disable=Y
```

Verify module parameter:

```bash
modinfo iwlwifi | grep lar_disable
```

Check regulatory and channel state:

```bash
iw reg get
iw phy
```

You may still manually set a regulatory domain:

```bash
sudo iw reg set JP
```

Note: cfg80211 regulatory enforcement is partially bypassed, so this may not behave as on stock systems.

---

## 📡 What is modified

### 1. iwlwifi (NVM parsing layer)

- `iwl-nvm-parse.c`
  - Overrides firmware-provided channel flags
  - Forces channels to be treated as valid and usable
  - Removes `NO_IR` restrictions
  - Modifies channel capability flags (bandwidth / usage)
  - Enables additional 6 GHz capabilities

### 2. iwlwifi (driver behavior)

- Adds `lar_disable` parameter
- Treats LAR as disabled when requested
- Prevents firmware-driven regulatory updates from restricting operation

### 3. cfg80211 / regulatory

- `net/wireless/reg.c`
  - Disables regulatory domain intersection
  - Ignores driver regulatory hints
  - Ignores Country IE updates
  - Allows non-signed regulatory database usage

---

## 🧩 How it works

This project modifies multiple enforcement layers:

| Layer          | Behavior                                         |
| -------------- | ------------------------------------------------ |
| Firmware (NVM) | Channel validity and capability flags overridden |
| iwlwifi        | LAR disabled and channel filtering bypassed      |
| cfg80211       | Regulatory merging and enforcement disabled      |

Key idea:

> Instead of trusting firmware/regulatory input, the driver forces channels and capabilities to be available.

---

## 🧪 Tested hardware

- Intel AX211

---

## ⚠️ Known limitations

- Firmware may still block actual RF transmission on some channels
- Displayed capabilities may exceed real hardware capability
- 5 GHz / 6 GHz AP operation may still fail depending on firmware
- AFC / VLP behavior may not fully match real regulatory requirements
- Behavior varies across firmware versions and chipsets
- Future kernel updates may break patch compatibility
