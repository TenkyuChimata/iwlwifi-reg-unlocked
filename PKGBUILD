pkgname=linux-wireless-reg-unlocked
_pkgbase=linux

_kernelpkgver=$(pacman -Q "${_pkgbase}" | awk '{print $2}')
_kernver=$(printf '%s\n' "${_kernelpkgver}" | sed 's/\.arch[0-9].*//')
_archrel=$(printf '%s\n' "${_kernelpkgver}" | sed -E "s/^${_kernver}\.(.*)$/\1/" | tr '.' '-')
_krel="${_kernver}-${_archrel}"

pkgver="${_kernver}"
pkgrel=4
pkgdesc="Patched iwlwifi and cfg80211 modules with LAR disable and modified regulatory behavior for Arch Linux"
arch=('x86_64')
url="https://github.com/TenkyuChimata/linux-wireless-reg-unlocked"
license=('GPL-2.0-only')

depends=(
  "${_pkgbase}=${_kernelpkgver}"
  "${_pkgbase}-headers=${_kernelpkgver}"
  'zstd'
)

optdepends=(
  'mkinitcpio: rebuild initramfs for mkinitcpio-based systems'
  'dracut: rebuild initramfs or UKI for dracut-based systems'
  'systemd: required for kernel-install based workflows'
  'sbctl: re-sign boot artifacts for Secure Boot systems if needed'
)

makedepends=(
  'bc'
  'kmod'
  'patch'
)

source=(
  "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-${_kernver}.tar.xz"
  '0001-iwlwifi-add-lar_disable.patch'
  '0002-iwlwifi-unlock-6ghz-and-noir.patch'
  '0003-cfg80211-disable-regdom-intersect-and-ignore-hints.patch'
  '0004-cfg80211-disable-regdb-signature-verification.patch'
  'iwlwifi-lar.conf'
  'dracut-wireless-reg-unlocked.conf'
)

sha256sums=(
  'c16068a3af12e3943dee3b1eef57ca70229c069128bfa1184fb3f48b219d55bf'
  'da2ab52ccdef2b93088c9e0c56bc1c166bf748d021b529cb2af2ff6c5d9e85cc'
  '8510a7a2b69f696999efddb40c79f3735049406d0f8432c2a23dd3f58ab8f883'
  'ef26436f30412184a8af4418f7deb00992ef7624d9dc4960d9730e56ed2cee25'
  '28f2defefbeaa0ec8cd1709981d1f5a31f3de9ae7ff178d0780cb26dd07195d5'
  'd0f468221c28f5f07a040f36df4dcf571d3931eef7ed273d4e57b631ef9540d3'
  '3758f059f40e24561f588829cb80384324d75b0745d5eba6f4b6313b5809e2d1'
)

install="${pkgname}.install"
options=(!debug)

prepare() {
  cd "${srcdir}/linux-${_kernver}"

  patch -Np1 -i "${srcdir}/0001-iwlwifi-add-lar_disable.patch"
  patch -Np1 -i "${srcdir}/0002-iwlwifi-unlock-6ghz-and-noir.patch"
  patch -Np1 -i "${srcdir}/0003-cfg80211-disable-regdom-intersect-and-ignore-hints.patch"
  patch -Np1 -i "${srcdir}/0004-cfg80211-disable-regdb-signature-verification.patch"

  # Fix relative source references for out-of-tree module builds
  if [[ -f drivers/net/wireless/intel/iwlwifi/dvm/Makefile ]]; then
    sed -i 's|$(srctree)/||g' drivers/net/wireless/intel/iwlwifi/dvm/Makefile
  fi

  if [[ -f drivers/net/wireless/intel/iwlwifi/mvm/Makefile ]]; then
    sed -i 's|$(srctree)/||g' drivers/net/wireless/intel/iwlwifi/mvm/Makefile
  fi

  if [[ -f drivers/net/wireless/intel/iwlwifi/mld/Makefile ]]; then
    sed -i 's|$(srctree)/||g' drivers/net/wireless/intel/iwlwifi/mld/Makefile
  fi
}

build() {
  local builddir="/usr/lib/modules/${_krel}/build"
  local iwlsrc="${srcdir}/linux-${_kernver}/drivers/net/wireless/intel/iwlwifi"
  local cfgsrc="${srcdir}/linux-${_kernver}/net/wireless"

  [[ -d "${builddir}" ]] || {
    echo "ERROR: Missing kernel build directory: ${builddir}"
    echo "Make sure ${_pkgbase}-headers=${_kernelpkgver} is installed."
    return 1
  }

  [[ -d "${iwlsrc}" ]] || {
    echo "ERROR: Missing iwlwifi source directory: ${iwlsrc}"
    return 1
  }

  [[ -d "${cfgsrc}" ]] || {
    echo "ERROR: Missing cfg80211 source directory: ${cfgsrc}"
    return 1
  }

  export KBUILD_BUILD_USER='builder'
  export KBUILD_BUILD_HOST='arch'
  export KBUILD_BUILD_VERSION='1'
  export KBUILD_BUILD_TIMESTAMP='1970-01-01'

  make -C "${builddir}" \
    M="${iwlsrc}" \
    KBUILD_BUILD_USER="${KBUILD_BUILD_USER}" \
    KBUILD_BUILD_HOST="${KBUILD_BUILD_HOST}" \
    KBUILD_BUILD_VERSION="${KBUILD_BUILD_VERSION}" \
    KBUILD_BUILD_TIMESTAMP="${KBUILD_BUILD_TIMESTAMP}" \
    modules

  make -C "${builddir}" \
    M="${cfgsrc}" \
    KBUILD_BUILD_USER="${KBUILD_BUILD_USER}" \
    KBUILD_BUILD_HOST="${KBUILD_BUILD_HOST}" \
    KBUILD_BUILD_VERSION="${KBUILD_BUILD_VERSION}" \
    KBUILD_BUILD_TIMESTAMP="${KBUILD_BUILD_TIMESTAMP}" \
    modules
}

package() {
  local moddir="${pkgdir}/usr/lib/modules/${_krel}/updates/${pkgname}"
  local iwlsrc="${srcdir}/linux-${_kernver}/drivers/net/wireless/intel/iwlwifi"
  local cfgsrc="${srcdir}/linux-${_kernver}/net/wireless"

  install -dm755 "${moddir}"

  # Core Intel wireless modules
  install -m644 "${iwlsrc}/iwlwifi.ko" "${moddir}/iwlwifi.ko"
  install -m644 "${iwlsrc}/mvm/iwlmvm.ko" "${moddir}/iwlmvm.ko"

  if [[ -f "${iwlsrc}/dvm/iwldvm.ko" ]]; then
    install -m644 "${iwlsrc}/dvm/iwldvm.ko" "${moddir}/iwldvm.ko"
  fi

  if [[ -f "${iwlsrc}/mld/iwlmld.ko" ]]; then
    install -m644 "${iwlsrc}/mld/iwlmld.ko" "${moddir}/iwlmld.ko"
  fi

  # Patched cfg80211
  install -m644 "${cfgsrc}/cfg80211.ko" "${moddir}/cfg80211.ko"

  # Strip debug symbols from packaged modules
  strip --strip-debug "${moddir}/iwlwifi.ko" || true
  strip --strip-debug "${moddir}/iwlmvm.ko" || true
  strip --strip-debug "${moddir}/cfg80211.ko" || true

  if [[ -f "${moddir}/iwldvm.ko" ]]; then
    strip --strip-debug "${moddir}/iwldvm.ko" || true
  fi

  if [[ -f "${moddir}/iwlmld.ko" ]]; then
    strip --strip-debug "${moddir}/iwlmld.ko" || true
  fi

  # modprobe configuration
  install -Dm644 "${srcdir}/iwlwifi-lar.conf" \
    "${pkgdir}/etc/modprobe.d/iwlwifi-lar.conf"

  # optional dracut configuration
  install -Dm644 "${srcdir}/dracut-wireless-reg-unlocked.conf" \
    "${pkgdir}/etc/dracut.conf.d/wireless-reg-unlocked.conf"

  # persist built kernel release for .install script
  install -Dm644 /dev/stdin \
    "${pkgdir}/usr/share/${pkgname}/kernel-version" <<EOF
${_krel}
EOF
}