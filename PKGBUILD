# Maintainer: Bruno Pagani <archange@archlinux.org>
# Contributor: Steef Hegeman <mail@steefhegeman.com>
# Contributor: Luca Weiss <luca (at) z3ntu (dot) xyz>
# Contributor: Julian Schacher <jspp@posteo.net>

_electron=electron22
pkgbase=element.io
pkgname=(element-web element-desktop)
pkgver=1.11.24
pkgrel=1
pkgdesc="Glossy Matrix collaboration client — "
arch=(x86_64)
url="https://element.io"
license=(Apache)
makedepends=(npm git yarn python rust tcl ${_electron} nodejs-lts-gallium libxcrypt-compat)
_url="https://github.com/vector-im/element"
source=(element-web-${pkgver}.tar.gz::${_url}-web/archive/v${pkgver}.tar.gz
        element-web-${pkgver}.tar.gz.asc::${_url}-web/releases/download/v${pkgver}/v${pkgver}-src.tar.gz.asc
        element-desktop-${pkgver}.tar.gz::${_url}-desktop/archive/v${pkgver}.tar.gz
        element-desktop-${pkgver}.tar.gz.asc::${_url}-desktop/releases/download/v${pkgver}/v${pkgver}-src.tar.gz.asc
        autolaunch.patch
        io.element.Element.desktop
        element-desktop.sh)
sha256sums=('52b333303d24bc57fe9ae3c7ace12ace0cdf337c745ede9677831b9aa64ce924'
            'SKIP'
            '3fddf18fe2d6c56f2ddafcaecb4aa6c09567d3f30af87a9e8c0267b342c1849d'
            'SKIP'
            'dccbbb5f8cecec627c1a3659e2696981904fe886373095f70ad8b77b05ad854d'
            '0103f28a32fe31f698836516783c1c70a76a0117b5df7fd0af5c422c224220f9'
            'c1bd9ace215e3ec9af14d7f28b163fc8c8b42e23a2cf04ce6f4ce2fcc465feba')
validpgpkeys=(712BFBEE92DCA45252DB17D7C7BE97EFA179B100) # Element Releases <releases@riot.im>

prepare() {
  # Specify electron version in launcher
  sed -i "s|@ELECTRON@|${_electron}|" element-desktop.sh

  cd element-web-${pkgver}
  yarn install --no-fund

  cd ../element-desktop-${pkgver}
  patch -p1 < ../autolaunch.patch
  sed -i 's|"target": "deb"|"target": "dir"|' package.json
  sed -i 's|"https://packages.element.io/desktop/update/"|null|' element.io/release/config.json
  yarn install --no-fund
}

build() {
  export NODE_OPTIONS=--openssl-legacy-provider
  cd element-web-${pkgver}
  VERSION=${pkgver} yarn build --offline

  cd ../element-desktop-${pkgver}
  export SQLCIPHER_STATIC=1
  yarn run build:native
  yarn run build
}

package_element-web() {
  pkgdesc+="web version."
  replaces=(riot-web vector-web)

  cd element-web-${pkgver}

  install -d "${pkgdir}"/{usr/share/webapps,etc/webapps}/element

  cp -r webapp/* "${pkgdir}"/usr/share/webapps/element/
  install -Dm644 config.sample.json -t "${pkgdir}"/etc/webapps/element/
  ln -s /etc/webapps/element/config.json "${pkgdir}"/usr/share/webapps/element/
  echo "${pkgver}" > "${pkgdir}"/usr/share/webapps/element/version
}

package_element-desktop() {
  pkgdesc+="desktop version."
  replaces=(riot-desktop)
  depends=("element-web=${pkgver}" ${_electron} libsecret)
  backup=('etc/element/config.json')

  cd element-desktop-${pkgver}

  install -d "${pkgdir}"{/usr/lib/element/,/etc/webapps/element}

  # Install the app content, replace the webapp with a symlink to the system package
  cp -r dist/linux-unpacked/resources/* "${pkgdir}"/usr/lib/element/
  ln -s /usr/share/webapps/element "${pkgdir}"/usr/lib/element/webapp

  # Config file
  ln -s /etc/element/config.json "${pkgdir}"/etc/webapps/element/config.json
  install -Dm644 element.io/release/config.json -t "${pkgdir}"/etc/element

  # Required extras
  install -Dm644 ../io.element.Element.desktop -t "${pkgdir}"/usr/share/applications/
  install -Dm755 ../${pkgname}.sh "${pkgdir}"/usr/bin/${pkgname}

  # Icons
  install -Dm644 ../element-web-${pkgver}/res/themes/element/img/logos/element-logo.svg "${pkgdir}"/usr/share/icons/hicolor/scalable/apps/io.element.Element.svg
  for i in 16 24 48 64 96 128 256 512; do
    install -Dm644 build/icons/${i}x${i}.png "${pkgdir}"/usr/share/icons/hicolor/${i}x${i}/apps/io.element.Element.png
  done
}
