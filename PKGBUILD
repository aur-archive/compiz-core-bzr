# Maintainer: Charles Bos <charlesbos1 AT gmail>
# Contributor: Rob McCathie <archaur at rmcc dot com dot au
# Contributor: Iven Hsu <ivenvd AT gmail>

pkgname=compiz-core-bzr
pkgver=3878
pkgrel=7
_bzrname=compiz
_bzrbranch=0.9.12
pkgdesc="Composite manager for Aiglx and Xgl, with plugins and CCSM (development version)"
arch=('i686' 'x86_64')
url="https://launchpad.net/compiz"
license=('GPL' 'LGPL' 'MIT')
depends=('boost' 'xorg-server' 'libxcomposite' 'startup-notification' 'librsvg' 'dbus' 'mesa' 'libxslt' 'fuse' 'glibmm' 'libxrender' 'libwnck' 'pygtk' 'desktop-file-utils' 'pyrex' 'protobuf' 'metacity2' 'glu' 'libsm' 'dconf')
makedepends=('cmake' 'bzr' 'intltool')
optdepends=(
  'gnome-control-center: GNOME keybindings support (need to rebuild this package)'
  'kdebase-workspace: KDE window decoration support (need to rebuild this package)'
  'kdebase-lib: KDE window decoration support (need to rebuild this package)'
  'automoc4: KDE window decoration support (need to rebuild this package)'
  'xorg-xprop: grab various window properties for use in window matching rules'
  'gnome-themes-standard: provides the Adwaita theme for gtk-window-decorator'
  'gconf: GConf Configuration Backend support (need to rebuild this package)'
)
conflicts=('compiz-core' 'compiz-core-devel')
provides=("compiz-core-devel=$_bzrbranch")
replaces=('compiz-bzr')
source=("$_bzrname::bzr+http://bazaar.launchpad.net/~compiz-team/$_bzrname/$_bzrbranch/"
        "place-plugin-fix.patch"
        "set-gwd-default.patch"
        "focus-prevention-disable.patch")
sha256sums=('SKIP'
            '36b883526a6922b87c4091f26be19bdaf595fc1e3464703075b5946dd9915d47'
            '3aa6cb70f357b3d34d51735f4b5bcb0479086d7c7336de4bd8157569d6c52c08'
            'f4897590b0f677ba34767a29822f8f922a750daf66e8adf47be89f7c2550cf4b')
install='compiz-core-bzr.install'

pkgver() {
  cd $_bzrname
  bzr revno
}

prepare() {
  cd $_bzrname

  # Fix broken placement menu in the place plugin
  patch -Np1 -i "$srcdir/place-plugin-fix.patch"

  # Set gtk-window-decorator as default in the Window Decoration plugin
  patch -Np1 -i "$srcdir/set-gwd-default.patch"

  # Set focus prevention level to off which means that new windows will always get focus
  patch -Np1 -i "$srcdir/focus-prevention-disable.patch"

  find -type f \( -name 'CMakeLists.txt' -or -name '*.cmake' \) -exec sed -e 's/COMMAND python/COMMAND python2/g' -i {} \;
  find compizconfig/ccsm -type f -exec sed -e 's|^#!.*python|#!/usr/bin/env python2|g' -i {} \;
}

build() {
  cd $_bzrname

  export PYTHON="/usr/bin/python2"

  mkdir build; cd build

  cmake .. \
    -DCMAKE_INSTALL_PREFIX="/usr" \
    -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 \
    -DPYTHON_LIBRARY=/usr/lib/libpython2.7.so \
    -DQT_QMAKE_EXECUTABLE=qmake-qt4 \
    -DCOMPIZ_DISABLE_SCHEMAS_INSTALL=On \
    -DCOMPIZ_BUILD_WITH_RPATH=Off \
    -DCOMPIZ_PACKAGING_ENABLED=On \
    -DBUILD_GTK=On \
    -DBUILD_METACITY=On \
    -DBUILD_KDE4=Off \
    -DUSE_GCONF=Off \
    -DUSE_GSETTINGS=On \
    -DCOMPIZ_BUILD_TESTING=Off \
    -DCOMPIZ_WERROR=Off \
    -DCOMPIZ_DEFAULT_PLUGINS="composite,opengl,decor,resize,place,move,ccp"

  make
}

package() {
  cd $_bzrname/build
  make DESTDIR="${pkgdir}" install

  # findcompiz_install needs COMPIZ_DESTDIR and install needs DESTDIR
  # make findcompiz_install
  CMAKE_DIR=$(cmake --system-information | grep '^CMAKE_ROOT' | awk -F\" '{print $2}')
  install -dm755 "${pkgdir}${CMAKE_DIR}/Modules/"
  install -m644 ../cmake/FindCompiz.cmake "${pkgdir}${CMAKE_DIR}/Modules/"	

  # Add documentation
  install -dm755 "${pkgdir}/usr/share/doc/compiz/"
  install ../{AUTHORS,NEWS,README} "${pkgdir}/usr/share/doc/compiz/"

  # Add gsettings schema files
  if ls generated/glib-2.0/schemas/ | grep -qm1 .gschema.xml; then
    install -dm755 "${pkgdir}/usr/share/glib-2.0/schemas/" 
    install -m644 generated/glib-2.0/schemas/*.gschema.xml "${pkgdir}/usr/share/glib-2.0/schemas/" 
  fi 

  # Remove GConf schemas
  rm -rv "${pkgdir}/usr/share/gconf/" 
}

