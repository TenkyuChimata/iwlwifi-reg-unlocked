# iwlwifi-reg-unlocked

Patched Intel **iwlwifi** kernel modules with **regulatory restrictions relaxed**, including:

- LAR (Location Aware Regulatory) disable
- 6 GHz channel exposure
- NO_IR (no initiate radiation) removal
- Forced 6 GHz VLP AP capability

Designed for **Arch Linux** users who need full control over Wi-Fi capabilities.

---

## 🚀 Features

- Disable Intel iwlwifi LAR restrictions
- Unlock hidden / restricted channels (including 6 GHz)
- Remove `NO_IR` limitations (allow AP/initiating transmissions)
- Enable 6 GHz VLP AP capability regardless of regulatory flags
- Preserve compatibility with Arch Linux kernel updates
- Built as external kernel modules (no full kernel rebuild)

---

## ⚠️ Disclaimer

> 🚨 **IMPORTANT**

This project **intentionally bypasses regulatory enforcement mechanisms**.

- May violate local wireless communication laws
- May cause interference with licensed spectrum users
- Not certified for production or commercial use

**By using this software, you accept full responsibility for compliance with your local regulations.**

---

## 📥 Installation

### Manual build

```bash
git clone https://github.com/TenkyuChimata/iwlwifi-reg-unlocked.git
cd iwlwifi-reg-unlocked
makepkg -si
```

---

## 🔧 Usage

After installation:

```bash
reboot
```

Set regulatory domain manually:

```bash
sudo iw reg set US
```

Verify:

```bash
iw reg get
```

---

## 📡 What is unlocked

This patch modifies iwlwifi behavior to:

### ✔ Channel handling

- Prevent 6 GHz channels from being discarded when LAR is disabled
- Allow channels without `NVM_CHANNEL_VALID`

### ✔ Transmission restrictions

- Remove `NO_IR` flags
- Allow initiating transmissions (AP / P2P GO)

### ✔ 6 GHz capabilities

- Force-enable `IEEE80211_CHAN_ALLOW_6GHZ_VLP_AP`
- Remove AFC/VLP client restrictions

### ✔ Regulatory bypass

- Ignore parts of regulatory capability (`reg_capa`)
- Allow manual override via `iw reg set`

---

## 🧩 How it works

This project patches:

- `iwl-nvm-parse.c`
- related regulatory flag handling paths

Key changes:

- Skip invalid channel filtering when LAR is disabled (for 6 GHz)
- Remove regulatory-based transmit restrictions
- Force-enable selected capabilities

---

## 🧪 Tested hardware

- Intel AX211

---

## ⚠️ Known limitations

- 6 GHz still depends on firmware capabilities
- Some AP modes may fail due to firmware constraints
- Future kernel updates may break patch offsets
