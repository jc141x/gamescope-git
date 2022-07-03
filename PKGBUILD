# Maintainer: daci <dac1@fedora.email>

_static_wlroots=1
_static_liftoff=1

_pkgname=gamescope
pkgname=${_pkgname}-git
pkgver=11b3ad772022.06.20
pkgrel=1
pkgdesc="Micro-compositor formerly known as steamcompmgr"
arch=(amd64)
url="https://github.com/Plagman/gamescope"
license=("custom:BSD-2-Clause")
depends=("libxcomposite1" "libxtst6" "libxres1" "libsdl2-2.0-0" "pipewire" "libseat1" "libxres1")
makedepends=("build-essential" "ca-certificates" "cmake" "file" "git" "glslang-tools" "libcap-dev" "libdrm-dev" "libgbm-dev" "libinput-dev" "libpipewire-0.3-dev" "libpixman-1-dev" "libsdl2-dev" "libseat-dev" "libstb-dev" "libsystemd-dev" "libvulkan-dev" "libwayland-dev" "libx11-dev" "libxcb-composite0-dev" "libxcb-icccm4-dev" "libxcb-res0-dev" "libxcomposite-dev" "libxdamage-dev" "libxkbcommon-dev" "libxrender-dev" "libxres-dev" "libxtst-dev" "libxxf86vm-dev" "meson" "pax-utils" "pkg-config" "wayland-protocols" "xwayland")
provides=($_pkgname "steamcompmgr")
conflicts=($_pkgname "steamcompmgr")
source=("git+https://github.com/Plagman/gamescope.git")
sha512sums=('SKIP')

if [ $_static_wlroots -gt 0 ]; then
    depends+=("libdrm-dev" "libxkbcommon-dev" "libinput-dev" "libpixman-1-dev" "xwayland" "libxcb-render-util0-dev" "libxcb-util-dev" "seatd")
    makedepends+=("cmake")
else
    depends+=("wlroots-git")
fi

if [ $_static_liftoff -gt 0 ]; then
    depends+=("libdrm-dev")
else
    depends+=("libliftoff<0.2.0")
fi

pkgver() {
    cd ${_pkgname}
    _always=$(git describe --always | sed -e 's:-:.:g' -e 's:v::')
    _commits=$(git rev-list --count HEAD | sed 's:-:.:g')
    _date=$(git log -1 --date=short --pretty=format:%cd)
    printf "%s%s%s\n" "${_commits}" "${_always}" "${_date}" | sed -e 's:-:.:g'  -e 's:_:.:g'
}

prepare() {
    cd "$srcdir/$_pkgname"

    for src in "${source[@]}"; do
        src="${src%%::*}"
        src="${src##*/}"
        [[ $src = *.patch ]] || continue
        echo "Applying patch $src..."
        git apply "../$src"
    done

    [ $_static_wlroots -gt 0 ] || rm -rf "subprojects/wlroots"
    [ $_static_liftoff -gt 0 ] || rm -rf "subprojects/libliftoff"
}

build() {

    _force_static=(stb)
    [ $_static_wlroots -gt 0 ] && _force_static+=(wlroots)
    [ $_static_liftoff -gt 0 ] && _force_static+=(libliftoff)

    # combine _force_static into comma (,) seperated list
    # warning: following code does not work when items in _force_static have spaces in them
    _force_fallback=$(echo "${_force_static[*]}" | tr " " ",")

    if [ -z "$_force_fallback" ]; then
        _force_fallback="[]"
    fi

    echo "Statically linking: $_force_fallback"

    meson setup --prefix /usr --buildtype=release --force-fallback-for=$_force_fallback "$srcdir/$_pkgname" build
    ninja -C build
}

check() {

    ninja -C build test
}

package() {

    DESTDIR="$pkgdir" ninja -C build install

    # Delete library files that were linked statically
    rm -rfv "$pkgdir/usr/include/wlr" "$pkgdir/usr/lib/libwlroots.a" "$pkgdir/usr/lib/libwlroots*" "$pkgdir/usr/lib/pkgconfig/wlroots.pc"
    rm -rfv "$pkgdir/usr/include/libliftoff.h" "$pkgdir/usr/lib/libliftoff.a" "$pkgdir/usr/lib/libliftoff*" "$pkgdir/usr/lib/pkgconfig/libliftoff.pc"

    # Delete empty directories
    find "$pkgdir" -type d -empty -print -delete

    cd "$srcdir/$_pkgname"

    install -Dm644 "LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
    install -Dm644 "README.md" "${pkgdir}/usr/share/doc/${_pkgname}/README.md"
}
