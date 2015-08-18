# $Id: PKGBUILD 194430 2013-09-16 10:39:31Z jgc $
# Maintainer: Jan de Groot <jgc@archlinux.org>
# Contributor: Alexander Baldeck <alexander@archlinux.org>

pkgname=xulrunner-current
pkgver=26.0
pkgrel=1
pkgdesc="Mozilla Runtime Environment"
arch=('i686' 'x86_64')
license=('MPL' 'GPL' 'LGPL')
depends=('gtk2' 'mozilla-common' 'nss>=3.15' 'libxt' 'libxrender' 'hunspell' 'startup-notification' 'mime-types' 'dbus-glib' 'alsa-lib' 'libevent' 'sqlite>=3.7.4' 'libvpx' 'python2')
makedepends=('zip' 'unzip' 'pkg-config' 'diffutils' 'yasm' 'mesa' 'autoconf2.13' 'nss>=3.15.3' 'nspr>=4.10.2')
url="http://wiki.mozilla.org/XUL:Xul_Runner"
source=("ftp://ftp.mozilla.org/pub/mozilla.org/xulrunner/releases/$pkgver/source/xulrunner-$pkgver.source.tar.bz2"
        'mozconfig'
        'mozilla-pkgconfig.patch'
        'shared-libs.patch')
# staticlibs is needed for some extensions to compile correctly
options=('!emptydirs' 'staticlibs')
replaces=('xulrunner-oss')
provides=('xulrunner=26.0')
conflicts=('xulrunner')
sha256sums=('2da60f50f5452975c565cef8a4343cb5424f81b39032e04822d01bb50b13340a'
            '3fba82b327f8825ebe93ceaeaea4968d57cf7d700f40bf4457b06d263bcc2e8f'
            '23485d937035648add27a7657f6934dc5b295e886cdb0506eebd02a43d07f269'
            'e2b4a00d14f4ba69c62b3f9ef9908263fbab179ba8004197cbc67edbd916fdf1')

prepare() {
  cd "$srcdir/mozilla-release"
  cp "$srcdir/mozconfig" .mozconfig

  #fix libdir/sdkdir - fedora
  patch -Np1 -i ../mozilla-pkgconfig.patch
  patch -Np1 -i ../shared-libs.patch

  # WebRTC build tries to execute "python" and expects Python 2
  # Workaround taken from chromium PKGBUILD
  mkdir -p "$srcdir/python2-path"
  ln -sf /usr/bin/python2 "$srcdir/python2-path/python"

  # configure script misdetects the preprocessor without an optimization level
  # https://bugs.archlinux.org/task/34644
  sed -i '/ac_cpp=/s/$CPPFLAGS/& -O2/' configure
}

build() {
  cd "$srcdir/mozilla-release"

  export PATH="$srcdir/python2-path:$PATH"
  export LDFLAGS="$LDFLAGS -Wl,-rpath,/usr/lib/xulrunner-$pkgver"
  export PYTHON="/usr/bin/python2"

  make -j1 -f client.mk build MOZ_MAKE_FLAGS="$MAKEFLAGS"
}

package() {
  cd "$srcdir/mozilla-release"
  make -j1 -f client.mk DESTDIR="$pkgdir" install


  # the main place where xulrunner is installed
  _maindestdir0="/usr/lib/xulrunner-$pkgver"
  _maindestdir="$pkgdir/$_maindestdir0"

  # replace their dictionaries with ours
  rm -rf "$_maindestdir"/{dictionaries,hyphenation}
  ln -sf /usr/share/hunspell "$_maindestdir/dictionaries"
  ln -sf /usr/share/hyphen "$_maindestdir/hyphenation"

  # add xulrunner library path to ld.so.conf
  install -d "$pkgdir/etc/ld.so.conf.d"
  echo "$_maindestdir0" > "$pkgdir/etc/ld.so.conf.d/xulrunner.conf"

  _develdestdir="$pkgdir/usr/lib/xulrunner-devel-$pkgver"
  chmod +x "$_develdestdir/sdk/bin/xpt.py"
  sed -i 's|!/usr/bin/env python$|!/usr/bin/env python2|' \
    "$_develdestdir"/sdk/bin/{xpt,header,typelib,xpidl}.py
}

