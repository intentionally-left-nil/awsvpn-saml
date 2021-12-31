# Maintainer: Anil Kulkarni (cd+awsvpn@terminal.space)
pkgname=awsvpn-saml
pkgver=2.5.5.r1.ge3bac09f
pkgrel=1
pkgdesc="Fork of openvpn configured to support conecting to an AWS vpn server with SAML authentication"
arch=('i686' 'x86_64')
url='https://github.com/AnilRedshift/awsvpn-saml'
license=('MIT')
depends=('openssl' 'lzo' 'lz4' 'systemd-libs' 'libsystemd.so' 'pkcs11-helper' 'libpkcs11-helper.so' 'bind-tools')
makedepends=('git' 'systemd' 'python-docutils' 'go')
source=(
  'git+https://github.com/OpenVPN/openvpn.git#branch=release/2.5'
  'git+https://github.com/samm-git/aws-vpn-client'
  'skip-broken-tests.patch'
  'main.go'
  'awsvpn'
  'LICENSE'
)
sha256sums=('SKIP'
            'SKIP'
            'f50f3a29c50fc1366e69c2c1e6a331459bfba70d76397e4f2b19e42dac8af9f1'
            'a67aaeef4ac97865d50c9c5e3e575f810637453cead738bfd2d844eda0d656c2'
            'eabcb2119489170a4011db67a5ce15970f72dba563139bb2a95397680a904b44'
            '4dc942c03bc14dc28fe9cb6d66f67c6374735a965ea1291916d91cb28d7e6fe5')

# Taken from openvpn-git's PKGBUILD
pkgver() {
  cd "${srcdir}/openvpn"

  if GITTAG="$(git describe --abbrev=0 --tags 2>/dev/null)"; then
    printf '%s.r%s.g%s' \
      "$(sed -e "s/^${pkgname%%-git}//" -e 's/^[-_/a-zA-Z]\+//' -e 's/[-_+]/./g' <<< "${GITTAG}")" \
      "$(git rev-list --count "${GITTAG}"..)" \
      "$(git rev-parse --short HEAD)"
  else
    printf '0.r%s.g%s' \
      "$(git rev-list --count master)" \
      "$(git rev-parse --short HEAD)"
  fi
}

prepare() {
  cd "${srcdir}/openvpn"
  # https://www.mail-archive.com/openvpn-devel@lists.sourceforge.net/msg19302.html
  sed -i '/^CONFIGURE_DEFINES=/s/set/env/g' configure.ac
  patch -Np1 < "${srcdir}/aws-vpn-client/openvpn-v2.5.1-aws.patch"
  patch -Np1 < "${srcdir}/skip-broken-tests.patch"
  autoreconf --force --install
}

build() { 
  mkdir -p "${srcdir}/build"
  cd "${srcdir}/build"
  "${srcdir}"/openvpn/configure \
    --prefix="/usr/share/${pkgname}" \
    --sbindir="/usr/share/${pkgname}/bin" \
    --enable-pkcs11 \
    --enable-plugins \
    --enable-systemd \
    --enable-x509-alt-username
  make

  go build -o "${srcdir}/build/awsvpnserver" "${srcdir}/main.go"
}

check() {
  cd "${srcdir}/build"

  make check
}

package() {
  cd "${srcdir}/build"
  # install openvpn to /usr/share/awsvpn-saml/bin/
  make DESTDIR="${pkgdir}" install
  install -D -m0755 ./awsvpnserver "${pkgdir}/usr/share/${pkgname}/bin/awsvpnserver"
  install -D -m0755 "${srcdir}/awsvpn" "${pkgdir}/usr/bin/awsvpn"

  # Add the licenses
  install -d -m0644 "${pkgdir}/usr/share/licenses/${pkgname}"
  ln -sf /usr/share/doc/openvpn/COPYING "${pkgdir}/usr/share/licenses/${pkgname}/openvpn-COPYING"
  ln -sf /usr/share/doc/openvpn/COPYRIGHT.GPL "${pkgdir}/usr/share/licenses/${pkgname}/openvpn-COPYRIGHT.GPL"
  install -D -m0644 "${srcdir}/aws-vpn-client/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/aws-vpn-client-LICENSE"
  install -D -m0644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
