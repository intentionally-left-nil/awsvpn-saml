# Maintainer: Anil Kulkarni (cd+awsvpn@terminal.space)

pkgname=awsvpn-saml
pkgver=2.5.1
pkgrel=1
pkgdesc="Fork of openvpn configured to support conecting to an AWS vpn server with SAML authentication"
arch=('i686' 'x86_64')
url='https://github.com/AnilRedshift/awsvpn-saml'
license=('MIT')
depends=('openssl' 'lzo' 'lz4' 'systemd-libs' 'libsystemd.so' 'pkcs11-helper' 'libpkcs11-helper.so' 'bind-tools')
makedepends=('git' 'systemd' 'python-docutils' 'go')
source=(
  "https://swupdate.openvpn.org/community/releases/openvpn-${pkgver}.tar.gz"
  'git+https://github.com/samm-git/aws-vpn-client'
  'skip-broken-tests-2.4.9.patch'
  'skip-broken-tests-2.5.1.patch'
  'main.go'
  'awsvpn'
  'LICENSE'
)
sha256sums=('e9582b8e9457994bd8d50012be82c23b2f465da51460c9b2360a81da0f4e06e6'
            'SKIP'
            '72f7d657f5525a62ff5d263e93e3ab210cbc44f5d11ed493c61e43c8e790df03'
            'f50f3a29c50fc1366e69c2c1e6a331459bfba70d76397e4f2b19e42dac8af9f1'
            'a67aaeef4ac97865d50c9c5e3e575f810637453cead738bfd2d844eda0d656c2'
            'a31ef5e3a6e3cd27bd3428eb71e4c98aa6c2c1aeb19644f1547870b0bc24f199'
            '4dc942c03bc14dc28fe9cb6d66f67c6374735a965ea1291916d91cb28d7e6fe5')

prepare() {
  cd "${srcdir}/openvpn-${pkgver}"
  # https://www.mail-archive.com/openvpn-devel@lists.sourceforge.net/msg19302.html
  sed -i '/^CONFIGURE_DEFINES=/s/set/env/g' configure.ac
  patch -Np1 < "${srcdir}/aws-vpn-client/openvpn-v${pkgver}-aws.patch"
  patch -Np1 < "${srcdir}/skip-broken-tests-${pkgver}.patch"
  autoreconf --force --install
}

build() { 
  mkdir -p "${srcdir}/build"
  cd "${srcdir}/build"
  "${srcdir}/openvpn-${pkgver}/configure" \
    --prefix="/usr/share/${pkgname}" \
    --sbindir="/usr/share/${pkgname}/bin"
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
