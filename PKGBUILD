pkgname=linux-wireless-reg-unlocked
_pkgbase=linux

_kernelpkgver=$(pacman -Q "${_pkgbase}" | awk '{print $2}')
_kernver=$(printf '%s\n' "${_kernelpkgver}" | sed 's/\.arch[0-9].*//')
_archrel=$(printf '%s\n' "${_kernelpkgver}" | sed -E "s/^${_kernver}\.(.*)$/\1/" | tr '.' '-')
_krel="${_kernver}-${_archrel}"

pkgver="${_kernver}"
pkgrel=1
pkgdesc="Patched Intel iwlwifi and cfg80211 modules with LAR disable, regulatory bypass and 6GHz/NO_IR unlock support for Arch Linux"
arch=('x86_64')
license=('GPL2')
url="https://github.com/TenkyuChimata/linux-wireless-reg-unlocked"
depends=(
  "${_pkgbase}=${_kernelpkgver}"
  "${_pkgbase}-headers=${_kernelpkgver}"
  'dracut'
  'systemd'
  'zstd'
)
makedepends=('bc' 'kmod')
source=(
  "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-${_kernver}.tar.xz"
  '0001-iwlwifi-add-lar_disable.patch'
  '0002-iwlwifi-unlock-6ghz-and-noir.patch'
  '0003-cfg80211-disable-regdom-intersect-and-ignore-hints.patch'
  'iwlwifi-lar.conf'
  'dracut-iwlwifi.conf'
)
sha256sums=(
  'SKIP'
  'da2ab52ccdef2b93088c9e0c56bc1c166bf748d021b529cb2af2ff6c5d9e85cc'
  '8510a7a2b69f696999efddb40c79f3735049406d0f8432c2a23dd3f58ab8f883'
  'ef26436f30412184a8af4418f7deb00992ef7624d9dc4960d9730e56ed2cee25'
  'd0f468221c28f5f07a040f36df4dcf571d3931eef7ed273d4e57b631ef9540d3'
  '92177e53d23660b4bef9cadf192418a6dead40f38606368fd44532a6a5272ca5'
)
install="${pkgname}.install"

options=(!strip !debug)

prepare() {
  cd "${srcdir}/linux-${_kernver}"

  patch -Np1 -i "${srcdir}/0001-iwlwifi-add-lar_disable.patch"
  patch -Np1 -i "${srcdir}/0002-iwlwifi-unlock-6ghz-and-noir.patch"
  patch -Np1 -i "${srcdir}/0003-cfg80211-disable-regdom-intersect-and-ignore-hints.patch"

  cd drivers/net/wireless/intel/iwlwifi
  [[ -f dvm/Makefile ]] && sed -i 's|$(srctree)/||g' dvm/Makefile
  [[ -f mvm/Makefile ]] && sed -i 's|$(srctree)/||g' mvm/Makefile
}

build() {
  local builddir="/usr/lib/modules/${_krel}/build"
  local iwlsrc="${srcdir}/linux-${_kernver}/drivers/net/wireless/intel/iwlwifi"
  local cfgsrcdir="${srcdir}/linux-${_kernver}/net/wireless"

  [[ -d "${builddir}" ]] || {
    echo "ERROR: Missing kernel build directory: ${builddir}"
    echo "Make sure ${_pkgbase}-headers=${_kernelpkgver} is installed."
    return 1
  }

  [[ -d "${iwlsrc}" ]] || {
    echo "ERROR: Missing iwlwifi source directory: ${iwlsrc}"
    return 1
  }

  [[ -d "${cfgsrcdir}" ]] || {
    echo "ERROR: Missing cfg80211 source directory: ${cfgsrcdir}"
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
    M="${cfgsrcdir}" \
    KBUILD_BUILD_USER="${KBUILD_BUILD_USER}" \
    KBUILD_BUILD_HOST="${KBUILD_BUILD_HOST}" \
    KBUILD_BUILD_VERSION="${KBUILD_BUILD_VERSION}" \
    KBUILD_BUILD_TIMESTAMP="${KBUILD_BUILD_TIMESTAMP}" \
    modules
}

package() {
  local moddir="${pkgdir}/usr/lib/modules/${_krel}/updates/iwlwifi-lar"
  local cfgdir="${pkgdir}/usr/lib/modules/${_krel}/updates/cfg80211"
  local srcsubdir="${srcdir}/linux-${_kernver}/drivers/net/wireless/intel/iwlwifi"
  local cfgsrc="${srcdir}/linux-${_kernver}/net/wireless/cfg80211.ko"

  install -dm755 "${moddir}"
  install -dm755 "${cfgdir}"

  install -m644 "${srcsubdir}/iwlwifi.ko" "${moddir}/iwlwifi.ko"
  install -m644 "${srcsubdir}/mvm/iwlmvm.ko" "${moddir}/iwlmvm.ko"

  if [[ -f "${srcsubdir}/dvm/iwldvm.ko" ]]; then
    install -m644 "${srcsubdir}/dvm/iwldvm.ko" "${moddir}/iwldvm.ko"
  fi

  if [[ -f "${cfgsrc}" ]]; then
    install -m644 "${cfgsrc}" "${cfgdir}/cfg80211.ko"
    strip --strip-debug "${cfgdir}/cfg80211.ko" || true
  fi

  strip --strip-debug "${moddir}/iwlwifi.ko" || true
  strip --strip-debug "${moddir}/iwlmvm.ko" || true

  if [[ -f "${moddir}/iwldvm.ko" ]]; then
    strip --strip-debug "${moddir}/iwldvm.ko" || true
  fi

  install -Dm644 "${srcdir}/iwlwifi-lar.conf" \
    "${pkgdir}/etc/modprobe.d/iwlwifi-lar.conf"

  install -Dm644 "${srcdir}/dracut-iwlwifi.conf" \
    "${pkgdir}/etc/dracut.conf.d/iwlwifi-lar.conf"

  install -Dm644 /dev/stdin \
    "${pkgdir}/usr/share/${pkgname}/kernel-version" <<EOF
${_krel}
EOF
}