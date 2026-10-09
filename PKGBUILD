# Maintainer: Dennis Maina <dennismyner7@gmail.com>
pkgname=littlenavmap-git
pkgver=3.0.18.r345.g9136dddc
_atoolsver=release/4.2
_marblever=lnm/1.2
_navmapver=release/3.2
_navconnectver=release/3.2
pkgrel=1
pkgdesc="Flight planner, navigation tool, moving map and airport information system for FSX, P3D, MSFS and X-Plane (git version)"
arch=(x86_64)
url="https://albar965.github.io/littlenavmap.html"
license=("GPL-3.0-or-later")
depends=(qt6-base qt6-svg qt6-declarative qt6-imageformats qt6-5compat curl glib2)
makedepends=(git cmake patchelf)
provides=(littlenavmap)
conflicts=(littlenavmap-bin littlenavmap)
source=(
    "atools::git+https://github.com/albar965/atools.git#branch=$_atoolsver"
    "marble::git+https://github.com/albar965/marble.git#branch=$_marblever"
    "littlenavconnect::git+https://github.com/albar965/littlenavconnect.git#branch=$_navconnectver"
    "littlenavmap::git+https://github.com/albar965/littlenavmap.git#branch=$_navmapver"
    "LittleNavmap.desktop"
    "0000-src_common_maptypes.h.patch"
    "0001-src_search_searchbasetable.cpp.patch"
    "0002_src_gui_timedialog.ui.patch"
    "0003_src_logbook_logdatadialog.ui.patch"
    "0004_src_routeexport_routeexportdialog.ui.patch"
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'a786cda60d4eb018130d586908b44135ec22b938af61e2e0031f82140980e5b9'
            '83d8bad51d53d7badf854aa94f33ce800aa8e5151070b584db9244358b75e719'
            'e9fecc5a510aa645be23f12390d777d7f5ea725ca04d00320f68dfcfaec5d43a'
            'e645836ec63a04e0dd3f7337bdd7707d93836872279b8355c1657b38c6a05872'
            'd9479c8316352a8a7c38df90f3cf596cb91c038c47ed743f55cb03b63025337d'
            '5f6f9d4220d0ecc945584e0022fb6224547e16eab9386930ab0c08a7982f011e')

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
}

package() {
    local _deploydir="$srcdir/deploy"
    local _approot="usr/lib/littlenavmap"

    install -d "$pkgdir/usr/bin" "$pkgdir/usr/lib"
    cp -r "$_deploydir" "$pkgdir/$_approot"

    install -Dm644 "$srcdir/LittleNavmap.desktop" "$pkgdir/usr/share/applications/littlenavmap.desktop"
    install -Dm644 "$_deploydir/Little Navmap/littlenavmap.svg" "$pkgdir/usr/share/icons/hicolor/scalable/apps/littlenavmap.svg"

    ln -sf "/$_approot/Little Navmap/littlenavmap" "$pkgdir/usr/bin/littlenavmap"
    ln -sf "/$_approot/Little Navconnect/littlenavconnect" "$pkgdir/usr/bin/littlenavconnect"
}
