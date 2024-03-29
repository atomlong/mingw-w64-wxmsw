# Maintainer: Ivan Puntiy <ivan.puntiy-at-gmail>
# Contributor: Schala

pkgname=mingw-w64-wxmsw
pkgver=3.0.5.1
pkgrel=2
pkgdesc="Win32 implementation of wxWidgets API for GUI (mingw-w64)"
arch=(any)
url="https://wxwidgets.org"
license=("custom:wxWindows")
makedepends=(mingw-w64-configure)
depends=(mingw-w64-crt mingw-w64-expat mingw-w64-libpng mingw-w64-libjpeg-turbo mingw-w64-libtiff)
options=(staticlibs !strip !buildflags)
conflicts=(mingw-w64-wxmsw2.9 mingw-w64-wxmsw-static)
provides=(mingw-w64-wxmsw2.9 mingw-w64-wxmsw-static)
source=("https://github.com/wxWidgets/wxWidgets/releases/download/v${pkgver}/wxWidgets-${pkgver}.tar.bz2"
        "fix-narrowing.patch"
        "cpp17-minmax.patch")
sha1sums=('406ac736f61d88a3a866aa501e01e408a642c6e7'
          '8657efb04e2c7befb83c3c9e5981f99b8484babf'
          'a427db2f11b4f69d6aaca5c840e7a9b43883f782')

_architectures="i686-w64-mingw32 x86_64-w64-mingw32"

# linking 64-bit shared libraries fails for me for an unknown reason, and I'm not sure if the problem is with compiler or my environment
# uncomment the following line to skip building it, if needed
_skip_shared_64bit=aye

prepare() {
  cd "${srcdir}/wxWidgets-${pkgver}"
  patch --forward --strip=1 --input="${srcdir}/fix-narrowing.patch"
  patch --forward --strip=1 --input="${srcdir}/cpp17-minmax.patch"
}

build() {
  local _build_flags="\
    	--with-msw \
    	--with-opengl \
    	--disable-mslu \
    	--enable-unicode \
    	--with-regex=builtin \
    	--disable-precomp-headers \
    	--enable-graphics_ctx \
    	--enable-webview \
    	--enable-mediactrl \
    	--with-libpng=sys \
    	--with-libxpm=builtin \
    	--with-libjpeg=sys \
    	--with-libtiff=sys"

  # Fix for current libuuid.a issues
  # see: https://github.com/Alexpux/MINGW-packages/issues/1761
  # looks like this was fixed, uncomment if needed
  # _build_flags="${_build_flags} LDFLAGS=-Wl,--allow-multiple-definition"

  cd "${srcdir}/wxWidgets-${pkgver}"
  for _arch in ${_architectures}; do
    # shared build
    if [[ -z "$_skip_shared_64bit" || ! ( $_arch =~ "x86_64" ) ]]; then
      mkdir -p build-shared-${_arch} && pushd build-shared-${_arch}
      ${_arch}-configure ${_build_flags} --enable-monolithic ..
      make
      popd
    fi

    # static build
    mkdir -p build-static-${_arch} && pushd build-static-${_arch}
    ${_arch}-configure ${_build_flags} --disable-shared ..
    make
    popd
  done
}

package() {
  mkdir -p "${pkgdir}/usr/bin"
  for _arch in ${_architectures}; do
    for _build in shared static; do
      if [[ -z "$_skip_shared_64bit" || $_build == "static" || ! ( $_arch =~ "x86_64" ) ]]; then
        cd "${srcdir}/wxWidgets-${pkgver}/build-${_build}-${_arch}"
        make DESTDIR="${pkgdir}" install
      fi
    done

    if [[ -z "$_skip_shared_64bit" || ! ( $_arch =~ "x86_64" ) ]]; then
      mv "${pkgdir}/usr/${_arch}/lib/"*.dll "${pkgdir}/usr/${_arch}/bin/"
      ${_arch}-strip --strip-unneeded "$pkgdir"/usr/${_arch}/bin/*.dll
    fi
    ${_arch}-strip -g "$pkgdir"/usr/${_arch}/lib/*.a

    ln -s "/usr/${_arch}/lib/wx/config/${_arch}-msw-unicode-${pkgver%.*}" \
       "$pkgdir/usr/bin/${_arch}-wx-config"

    # rm "${pkgdir}/usr/${_arch}/bin/"*.exe
    # rm "$pkgdir/usr/${_arch}/bin/wxrc-3.0"
    # rm -r "$pkgdir/usr/${_arch}/share"
  done
}
