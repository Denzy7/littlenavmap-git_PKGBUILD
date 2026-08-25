# Maintainer: Dennis Maina <dennismyner7@gmail.com>
pkgname=littlenavmap-git
pkgver=3.0.18.r336.gbf08619
_atoolsver=release/4.2
_marblever=lnm/1.2
_navmapver=release/3.2
_navconnectver=release/3.2
pkgrel=1
epoch=
pkgdesc="A Free Open Source Flight Planner, Navigation Tool, Moving
Map, Airport Search, and Airport Information System for
Flight Simulator X, Prepar3D, Microsoft Flight Simulator
2020, and X-Plane (git version)"
arch=(x86_64)
url="https://albar965.github.io/littlenavmap.html"
license=("GPL3")
groups=()
depends=(qt6-base qt6-svg qt6-declarative qt6-imageformats qt6-5compat curl glib2)
makedepends=(git cmake patchelf)
checkdepends=()
optdepends=()
provides=()
conflicts=(littlenavmap-bin littlenavmap)
replaces=()
backup=()
options=()
install=
changelog=
source=(
    "LittleNavmap.desktop"
    "0000-src_common_maptypes.h.patch"
    "0001-src_search_searchbasetable.cpp.patch"
    "0002_src_gui_timedialog.ui.patch"
    "0003_src_logbook_logdatadialog.ui.patch"
    "0004_src_routeexport_routeexportdialog.ui.patch"
    "0005-src_common_formatter.cpp.patch"
)
noextract=()
sha256sums=('cd8009076d1f6c4300f98089eeb8acb5ea09fdb3bbafe83ccce910f0a05da51d'
            '83d8bad51d53d7badf854aa94f33ce800aa8e5151070b584db9244358b75e719'
            'e9fecc5a510aa645be23f12390d777d7f5ea725ca04d00320f68dfcfaec5d43a'
            'e645836ec63a04e0dd3f7337bdd7707d93836872279b8355c1657b38c6a05872'
            'd9479c8316352a8a7c38df90f3cf596cb91c038c47ed743f55cb03b63025337d'
            '5f6f9d4220d0ecc945584e0022fb6224547e16eab9386930ab0c08a7982f011e'
            '8754e4b7cf7e88c2556fda52142692ed8a1f3521160d46682d719f92ade56f63')
validpgpkeys=()

pkgver() {
    cd littlenavmap
    git describe --long --tags | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
    # THESE SHOULD BE GLOBAL VARIABLES BUT makepkg seems to not have $srcdir available
    # outside of the functions??
    _builddir="$srcdir/$pkgname-$pkgver-builddir"
    _deploydir="$srcdir/deploy"

    mkdir -p "$_builddir"
    mkdir -p "$_deploydir"
    sed -i "s/@VERSION@/$pkgver/" "$srcdir/LittleNavmap.desktop"

    if [[ -d "$srcdir/atools/.git" ]]; then
        git -C "$srcdir/atools" pull
    else
        git -C "$srcdir" clone --branch $_atoolsver --depth 1 https://github.com/albar965/atools.git
    fi

    if [[ -d "$srcdir/marble/.git" ]]; then
        git -C "$srcdir/marble" pull
    else
        git -C "$srcdir" clone --branch $_marblever --depth 1 https://github.com/albar965/marble.git
    fi

    if [[ -d "$srcdir/littlenavconnect/.git" ]]; then
        git -C "$srcdir/littlenavconnect" pull
    else
        git -C "$srcdir" clone --branch $_navconnectver --depth 1 https://github.com/albar965/littlenavconnect.git
    fi

    if [[ -d "$srcdir/littlenavmap/.git" ]]; then
        pushd "$srcdir/littlenavmap"

        git restore .
        git pull
    else
        git -C "$srcdir" clone --branch $_navmapver --depth 1 https://github.com/albar965/littlenavmap.git
        pushd "$srcdir/littlenavmap"
        while ! git describe --tags --long >/dev/null 2>&1; do
            echo "no tag found. deepening"
            git fetch --deepen=100 origin
        done
    fi

    echo "patching sources"
    for patch in "$srcdir"/*.patch; do
        git apply "$patch"
    done
    popd
}

build() {
    # THESE SHOULD BE GLOBAL VARIABLES BUT makepkg seems to not have $srcdir available
    # outside of the functions??
    _builddir="$srcdir/$pkgname-$pkgver-builddir"
    _deploydir="$srcdir/deploy"

    _atools_srcdir="$srcdir/atools"
    _atools_builddir="$_builddir/atools"

    _marble_srcdir="$srcdir/marble"
    _marble_builddir="$_builddir/marble"
    _marble_installdir="$_marble_builddir/marble-rel"

    _littlenavmap_srcdir="$srcdir/littlenavmap"
    _littlenavmap_builddir="$_builddir/littlenavmap"

    _littlenavconnect_srcdir="$srcdir/littlenavconnect"
    _littlenavconnect_builddir="$_builddir/littlenavconnect"

    # atools
    mkdir -p "$_atools_builddir"
    cd "$_atools_builddir"
    ATOOLS_NO_CRASHHANDLER=true qmake6 "$_atools_srcdir/atools.pro" -spec linux-g++ CONFIG+=release
    make -j$(nproc)

    # marble
    mkdir -p "$_marble_builddir"
    cd "$_marble_builddir"
    #TODO: remove old cmake
    CMAKE_POLICY_VERSION_MINIMUM=3.5 cmake -S "$_marble_srcdir" -B . -DCMAKE_BUILD_TYPE=Release -DSTATIC_BUILD=TRUE -DQTONLY=TRUE -DBUILD_MARBLE_EXAMPLES=NO -DBUILD_INHIBIT_SCREENSAVER_PLUGIN=NO -DBUILD_MARBLE_APPS=NO -DBUILD_MARBLE_EXAMPLES=NO -DBUILD_MARBLE_TESTS=NO -DBUILD_MARBLE_TOOLS=NO -DBUILD_TESTING=NO -DBUILD_WITH_DBUS=NO -DMARBLE_EMPTY_MAPTHEME=YES -DMOBILE=NO -DWITH_DESIGNER_PLUGIN=NO -DWITH_Phonon=NO -DWITH_Qt5Location=NO -DWITH_Qt5Positioning=NO -DWITH_Qt5SerialPort=NO -DWITH_ZLIB=NO -DWITH_libgps=NO -DWITH_libshp=NO -DWITH_libwlocate=NO -DCMAKE_INSTALL_PREFIX="$_marble_installdir" -DEXEC_INSTALL_PREFIX="$_marble_installdir"
    make -j$(nproc)
    make install

    # littlenavmap
    mkdir -p "$_littlenavmap_builddir"
    cd "$_littlenavmap_builddir"
    ATOOLS_NO_CRASHHANDLER=true ATOOLS_INC_PATH="$_atools_srcdir/src" ATOOLS_LIB_PATH="$_atools_builddir" MARBLE_INC_PATH="$_marble_installdir/include" DEPLOY_BASE="$_deploydir" MARBLE_LIB_PATH="$_marble_installdir/lib" qmake6 "$_littlenavmap_srcdir/littlenavmap.pro" -spec linux-g++ CONFIG+=release
    make copydata
    # qmake shenanigans adding old call
    # changes may trigger a whole recompilation. maybe check already commented?
    grep -rlF --include='*.h' 'setTimeSpec' "$_littlenavmap_builddir" |
        while IFS= read -r file; do
            sed -i '/setTimeSpec/s|^|//|' "$file"
        done
    make -j$(nproc)
    make deploy

    #cleanup
    eval rm -fr "$_deploydir/Little\ Navmap/lib/{libQt*,libicu*,iconengines,imageformats,platform*,printsupport,sqldrivers}"

    # littlanavconnect
    mkdir -p "$_littlenavconnect_builddir"
    cd "$_littlenavconnect_builddir"
    ATOOLS_NO_CRASHHANDLER=true ATOOLS_INC_PATH="$_atools_srcdir/src" ATOOLS_LIB_PATH="$_atools_builddir" DEPLOY_BASE="$_deploydir" qmake6 "$_littlenavconnect_srcdir/littlenavconnect.pro" -spec linux-g++ CONFIG+=release
    make -j$(nproc)
    make deploy
    eval rm -fr "$_deploydir/Little\ Navconnect/lib"
    eval rm "$_deploydir/Little\ Navconnect/qt.conf"
    eval rm "$_deploydir/Little\ Navmap/qt.conf"
}

package() {
    # THESE SHOULD BE GLOBAL VARIABLES BUT makepkg seems to not have $srcdir available
    # outside of the functions??
    _deploydir="$srcdir/deploy"
    _approot="usr/lib/littlenavmap-$pkgver"
    _approot_pkg="$pkgdir/$_approot"

    mkdir -p "${pkgdir}/usr/bin"
    mkdir -p "${pkgdir}/usr/lib"
    mkdir -p "${pkgdir}/usr/share/applications"

    cp -r "$_deploydir" "$_approot_pkg"

    cp "$srcdir/LittleNavmap.desktop" "${pkgdir}/usr/share/applications"

    ln -sf "/$_approot/Little Navmap/littlenavmap" "${pkgdir}/usr/bin/littlenavmap"
    ln -sf "/$_approot/Little Navconnect/littlenavconnect" "${pkgdir}/usr/bin/littlenavconnect"
}

