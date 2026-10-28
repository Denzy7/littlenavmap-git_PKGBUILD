# Maintainer: Dennis Maina <dennismyner7@gmail.com>
pkgname=littlenavmap-git
pkgver=3.2.2.beta.r7.g518988ce
_atoolsver=release/4.2
_marblever=lnm/1.2
_navmapver=release/3.2
_navconnectver=release/3.2
_xpconnectver=release/1.4
_xpsdkver=430
pkgrel=2
pkgdesc="Flight planner, navigation tool, moving map and airport information system for FSX, P3D, MSFS and X-Plane (git version)"
arch=(x86_64)
url="https://albar965.github.io/littlenavmap.html"
license=("GPL-3.0-or-later")
depends=(qt6-base qt6-svg qt6-declarative qt6-imageformats qt6-5compat curl glib2)
makedepends=(git cmake patchelf littlenavmap-qt6-base-static)
provides=(littlenavmap)
conflicts=(littlenavmap-bin littlenavmap)
source=(
    "atools::git+https://github.com/albar965/atools.git#branch=$_atoolsver"
    "marble::git+https://github.com/albar965/marble.git#branch=$_marblever"
    "littlenavconnect::git+https://github.com/albar965/littlenavconnect.git#branch=$_navconnectver"
    "littlenavmap::git+https://github.com/albar965/littlenavmap.git#branch=$_navmapver"
    "littlexpconnect::git+https://github.com/albar965/littlexpconnect.git#branch=$_xpconnectver"
    "https://developer.x-plane.com/wp-content/plugins/code-sample-generation/sdk_zip_files/XPSDK$_xpsdkver.zip"
    "LittleNavmap.desktop"
    "0001-src_search_searchbasetable.cpp.patch"
    "0002_src_gui_timedialog.ui.patch"
    "0003_src_logbook_logdatadialog.ui.patch"
    "0004_src_routeexport_routeexportdialog.ui.patch"
    "0005-src_webapi_abstractlnmactionscontroller.h.patch"
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'b9875ab27b593927b4f9b3e0ddfffe7401ee5dce6d86b50aea0da65f70ff7816'
            'a786cda60d4eb018130d586908b44135ec22b938af61e2e0031f82140980e5b9'
            'e9fecc5a510aa645be23f12390d777d7f5ea725ca04d00320f68dfcfaec5d43a'
            'e645836ec63a04e0dd3f7337bdd7707d93836872279b8355c1657b38c6a05872'
            'd9479c8316352a8a7c38df90f3cf596cb91c038c47ed743f55cb03b63025337d'
            '5f6f9d4220d0ecc945584e0022fb6224547e16eab9386930ab0c08a7982f011e'
            '47960be40b36806639af8645151d4dddd7a39f3e5b38f720789ae994d354ecf4')

# makepkg runs gdb-add-index on every binary for the debug package. gdb 18.1
# cleans up with an unquoted `rm -f $tmp_files`, so with spaces in the path
# ("Little Navmap") its temporary index files end up in the package. makepkg
# calls this function instead of the binary; drop it once gdb is fixed
gdb-add-index() {
    local _ret=0
    command gdb-add-index "$@" || _ret=$?
    rm -f -- "${@: -1}".{gdb-index,debug_names,debug_str,debug_str.merge,debug_str.err}
    return $_ret
}

pkgver() {
    cd littlenavmap
    git describe --long --tags | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
    # makepkg force-checks out the git sources on every run, so the patches
    # always apply to pristine files
    local _patch
    for _patch in "${source[@]}"; do
        [[ $_patch == *.patch ]] || continue
        echo "applying $_patch"
        git -C "$srcdir/littlenavmap" apply "$srcdir/$_patch"
    done
}

build() {
    # fixed (unversioned) build dir so incremental rebuilds can reuse it
    local _builddir="$srcdir/build"
    local _deploydir="$srcdir/deploy"

    local _atools_srcdir="$srcdir/atools"
    local _atools_builddir="$_builddir/atools"

    local _marble_srcdir="$srcdir/marble"
    local _marble_builddir="$_builddir/marble"
    local _marble_installdir="$_marble_builddir/marble-rel"

    local _littlenavmap_srcdir="$srcdir/littlenavmap"
    local _littlenavmap_builddir="$_builddir/littlenavmap"

    local _littlenavconnect_srcdir="$srcdir/littlenavconnect"
    local _littlenavconnect_builddir="$_builddir/littlenavconnect"

    # qmake ignores CFLAGS/CXXFLAGS/LDFLAGS, pass makepkg's flags explicitly
    local _qmake_flags=(
        -spec linux-g++ CONFIG+=release
        "QMAKE_CFLAGS_RELEASE=$CFLAGS"
        "QMAKE_CXXFLAGS_RELEASE=$CXXFLAGS"
        "QMAKE_LFLAGS_RELEASE=$LDFLAGS"
    )

    # start from a clean deploy dir so nothing stale gets packaged
    rm -rf "$_deploydir"
    mkdir -p "$_builddir" "$_deploydir"

    # atools
    mkdir -p "$_atools_builddir"
    cd "$_atools_builddir"
    ATOOLS_NO_CRASHHANDLER=true qmake6 "$_atools_srcdir/atools.pro" "${_qmake_flags[@]}"
    make -j$(nproc)

    # marble
    mkdir -p "$_marble_builddir"
    cd "$_marble_builddir"
    #TODO: remove old cmake
    CMAKE_POLICY_VERSION_MINIMUM=3.5 cmake -S "$_marble_srcdir" -B . -DCMAKE_BUILD_TYPE=Release -DSTATIC_BUILD=TRUE -DQTONLY=TRUE -DBUILD_MARBLE_EXAMPLES=NO -DBUILD_INHIBIT_SCREENSAVER_PLUGIN=NO -DBUILD_MARBLE_APPS=NO -DBUILD_MARBLE_TESTS=NO -DBUILD_MARBLE_TOOLS=NO -DBUILD_TESTING=NO -DBUILD_WITH_DBUS=NO -DMARBLE_EMPTY_MAPTHEME=YES -DMOBILE=NO -DWITH_DESIGNER_PLUGIN=NO -DWITH_Phonon=NO -DWITH_Qt5Location=NO -DWITH_Qt5Positioning=NO -DWITH_Qt5SerialPort=NO -DWITH_ZLIB=NO -DWITH_libgps=NO -DWITH_libshp=NO -DWITH_libwlocate=NO -DCMAKE_INSTALL_PREFIX="$_marble_installdir" -DEXEC_INSTALL_PREFIX="$_marble_installdir"
    make -j$(nproc)
    make install

    # littlenavmap
    mkdir -p "$_littlenavmap_builddir"
    cd "$_littlenavmap_builddir"
    ATOOLS_NO_CRASHHANDLER=true ATOOLS_INC_PATH="$_atools_srcdir/src" ATOOLS_LIB_PATH="$_atools_builddir" MARBLE_INC_PATH="$_marble_installdir/include" DEPLOY_BASE="$_deploydir" MARBLE_LIB_PATH="$_marble_installdir/lib" qmake6 "$_littlenavmap_srcdir/littlenavmap.pro" "${_qmake_flags[@]}"
    make copydata
    # qmake shenanigans adding old call. only touch headers that still have an
    # uncommented setTimeSpec so reruns don't stack // or trigger recompiles
    grep -rlP --include='*.h' '^(?!\s*//).*setTimeSpec' "$_littlenavmap_builddir" |
        while IFS= read -r file; do
            sed -i -E '/^\s*\/\//!{/setTimeSpec/s|^|//|}' "$file"
        done
    make -j$(nproc)
    make deploy

    #cleanup
    rm -rf "$_deploydir/Little Navmap/lib/"{libQt*,libicu*,iconengines,imageformats,platform*,printsupport,sqldrivers}

    # littlenavconnect
    mkdir -p "$_littlenavconnect_builddir"
    cd "$_littlenavconnect_builddir"
    ATOOLS_NO_CRASHHANDLER=true ATOOLS_INC_PATH="$_atools_srcdir/src" ATOOLS_LIB_PATH="$_atools_builddir" DEPLOY_BASE="$_deploydir" qmake6 "$_littlenavconnect_srcdir/littlenavconnect.pro" "${_qmake_flags[@]}"
    make -j$(nproc)
    make deploy
    rm -rf "$_deploydir/Little Navconnect/lib"
    rm -f "$_deploydir/Little Navconnect/qt.conf" "$_deploydir/Little Navmap/qt.conf"

    # littlexpconnect (X-Plane plugin). Like upstream's build_release.sh it is
    # built against a static Qt and a stripped down atools, so lin.xpl carries
    # its own hidden QtCore and only needs glibc
    local _qmake_static=/opt/littlenavmap-qt6-base-static/bin/qmake
    local _atools_static_builddir="$_builddir/atools-static"
    local _littlexpconnect_srcdir="$srcdir/littlexpconnect"
    local _littlexpconnect_builddir="$_builddir/littlexpconnect"
    local _atools_static_env=(
        ATOOLS_NO_FS=true ATOOLS_NO_GRIB=true ATOOLS_NO_GUI=true
        ATOOLS_NO_ROUTING=true ATOOLS_NO_SQL=true ATOOLS_NO_TRACK=true
        ATOOLS_NO_USERDATA=true ATOOLS_NO_WEATHER=true ATOOLS_NO_WEB=true
        ATOOLS_NO_WMM=true ATOOLS_NO_NAVSERVER=true ATOOLS_NO_CRASHHANDLER=true
        ATOOLS_NO_QT5COMPAT=true
    )

    # atools (static Qt)
    mkdir -p "$_atools_static_builddir"
    cd "$_atools_static_builddir"
    env "${_atools_static_env[@]}" "$_qmake_static" "$_atools_srcdir/atools.pro" "${_qmake_flags[@]}"
    make -j$(nproc)

    # littlexpconnect
    mkdir -p "$_littlexpconnect_builddir"
    cd "$_littlexpconnect_builddir"
    env "${_atools_static_env[@]}" ATOOLS_INC_PATH="$_atools_srcdir/src" ATOOLS_LIB_PATH="$_atools_static_builddir" XPSDK_BASE="$srcdir/SDK" DEPLOY_BASE="$_deploydir" "$_qmake_static" "$_littlexpconnect_srcdir/littlexpconnect.pro" "${_qmake_flags[@]}"
    make -j$(nproc)
    make deploy
    # like upstream's pack_deploy.sh, ship it inside the Little Navmap folder:
    # Tools > Install Little Xpconnect is only enabled when it's found there
    mv "$_deploydir/Little Xpconnect" "$_deploydir/Little Navmap/"
}

package() {
    local _deploydir="$srcdir/deploy"
    local _approot="usr/lib/littlenavmap"

    install -d "$pkgdir/usr/bin" "$pkgdir/usr/lib"
    cp -r "$_deploydir" "$pkgdir/$_approot"

    install -Dm644 "$srcdir/LittleNavmap.desktop" "$pkgdir/usr/share/applications/littlenavmap.desktop"
    install -Dm644 "$_deploydir/Little Navmap/littlenavmap.svg" "$pkgdir/usr/share/icons/hicolor/scalable/apps/littlenavmap.svg"

    # wrappers instead of symlinks: started through PATH, Qt resolves argv[0]
    # to /usr/bin without following the symlink, so applicationDirPath()
    # (map themes, data, help, Little Xpconnect) pointed to /usr/bin
    local _bin
    for _bin in "Little Navmap/littlenavmap" "Little Navconnect/littlenavconnect"; do
        printf '#!/bin/sh\nexec "%s" "$@"\n' "/$_approot/$_bin" > "$pkgdir/usr/bin/${_bin##*/}"
        chmod 755 "$pkgdir/usr/bin/${_bin##*/}"
    done
}
