# iwlwifi-reg-unlocked

Patched Intel **iwlwifi** kernel modules with **regulatory restrictions relaxed**, including:

- LAR (Location Aware Regulatory) disable
- 6 GHz channel exposure
- NO_IR (no initiate radiation) removal
- Forced 6 GHz VLP AP capability

Designed for **Arch Linux** users who need full control over Wi-Fi capabilities.

---

## 📦 AUR

> ⚠️ Package name may differ depending on release branch

https://aur.archlinux.org/packages/iwlwifi-lar-patched

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

### Using an AUR helper (recommended)

```bash
yay -S iwlwifi-lar-patched
```

### Manual build

```bash
git clone https://aur.archlinux.org/iwlwifi-lar-patched.git
cd iwlwifi-lar-patched
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

## 🔐 Secure Boot

If Secure Boot is enabled, kernel modules must be signed.

Example:

```bash
./sign-modules.sh
```

Ensure your keys are enrolled (e.g. using `sbctl`).

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

## 🔄 Updating

This package follows Arch kernel updates.

You must rebuild after kernel upgrades:

```bash
yay -Syu
```

or manually rebuild.

---

## 🧪 Tested hardware

- Intel AX200
- Intel AX210
- Intel AX211

---

## ⚠️ Known limitations

- 6 GHz still depends on firmware capabilities
- Some AP modes may fail due to firmware constraints
- Future kernel updates may break patch offsets

---

## 📁 Repository Structure

- `PKGBUILD` – Arch package definition
- `.SRCINFO` – AUR metadata
- `0001-*.patch` – LAR disable patch
- `0002-*.patch` – regulatory unlock patch
- `sign-modules.sh` – Secure Boot helper

---

## 📜 License

This project follows the original Linux kernel licensing (GPLv2).

---

## ❤️ Credits

- Intel iwlwifi driver developers
- Linux wireless subsystem maintainers
- Arch Linux community
- Reverse engineering & patch contributors

---

## 🐾 Maintainer

TenkyuChimata
