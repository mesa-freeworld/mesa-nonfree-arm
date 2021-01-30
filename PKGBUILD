# Maintainer: Dan Johansen <strit@manjaro.org>
# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Maintainer: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>

# ALARM: Kevin Mihelich <kevin@archlinuxarm.org>
#  - Removed DRI and Gallium3D drivers/packages for chipsets that don't exist in our ARM devices (intel, VMware svga).
#  - disable assembly and rip out VC4 forced NEON for v6/v7
#  - remove makedepend on valgrind, -Dvalgrind=false

pkgbase=mesa
pkgname=('vulkan-mesa-layers' 'opencl-mesa' 'vulkan-radeon' 'vulkan-swrast' 'vulkan-broadcom' 'libva-mesa-driver' 'mesa-vdpau' 'mesa')
pkgdesc="An open-source implementation of the OpenGL specification"
pkgver=20.3.4
pkgrel=1
arch=('x86_64' 'aarch64')
makedepends=('python-mako' 'libxml2' 'libx11' 'xorgproto' 'libdrm' 'libxshmfence' 'libxxf86vm'
             'libxdamage' 'libvdpau' 'libva' 'wayland' 'wayland-protocols' 'zstd' 'elfutils' 'llvm'
             'libomxil-bellagio' 'libclc' 'clang' 'libglvnd' 'libunwind' 'lm_sensors' 'libxrandr'
             'glslang' 'vulkan-icd-loader' 'meson')
url="https://www.mesa3d.org/"
license=('custom')
source=(https://mesa.freedesktop.org/archive/mesa-${pkgver}.tar.xz
        0001-Rip-out-VC4-forced-NEON.patch
        0002-revert-glx-Implement-GLX_EXT_swap_control-for-DRI2-a.patch
        LICENSE)
sha512sums=('81c4d032213b4aef842f1594e0e89bc0045f7ca7ce5f267b62a0f8236eb12ab09c1f780d8b3776b3072f37cd0bd8829f8a1330a749ccf462471b262ef8097477'
            'ba55fd9816ebd9147be120da1fd4fa0364d19967a11570e6d5dd9d8b4f7971df46ced8b151ee07afaaa98043e131eed14918ec25f8c9b0f7e5c53f452674ee5c'
            '7f15f5020de655b99e63e47b29ff03e9025c3004844e5b3a95a7da83043a5806f6d7d256e81b9cdb97d082e2526f6f68b4fdfb4a63c12bb17a6c8ed7368d6a86'
            'f9f0d0ccf166fe6cb684478b6f1e1ab1f2850431c06aa041738563eb1808a004e52cdec823c103c9e180f03ffc083e95974d291353f0220fe52ae6d4897fecc7')

prepare() {
  cd mesa-$pkgver

  # Apply patches
  patch -Np1 -i "${srcdir}/0002-revert-glx-Implement-GLX_EXT_swap_control-for-DRI2-a.patch"
  
  #[[ $CARCH == "armv6h" || $CARCH == "armv7h" ]] && patch -p1 -i ../0001-Rip-out-VC4-forced-NEON.patch || true
}

build() {
  MESON_OPT="-D asm=false"
  case "${CARCH}" in
    armv6h)  GALLIUM=",vc4" ;;
    armv7h)  GALLIUM=",etnaviv,kmsro,lima,panfrost,tegra,v3d,vc4" ;;
    aarch64) GALLIUM=",etnaviv,kmsro,lima,panfrost,v3d,vc4" ;;
  esac

  arch-meson mesa-$pkgver build \
    -D b_lto=false \
    -D b_ndebug=true \
    -D platforms=x11,wayland \
    -D dri-drivers=r100,r200,nouveau \
    -D gallium-drivers=r300,r600,radeonsi,freedreno,nouveau,swrast,virgl,zink${GALLIUM} \
    -D vulkan-drivers=amd,swrast,broadcom \
    -D vulkan-overlay-layer=true \
    -D vulkan-device-select-layer=true \
    -D dri3=enabled \
    -D egl=enabled \
    -D gallium-extra-hud=true \
    -D gallium-nine=true \
    -D gallium-omx=bellagio \
    -D gallium-opencl=icd \
    -D gallium-va=enabled \
    -D gallium-vdpau=enabled \
    -D gallium-xa=disabled \
    -D gallium-xvmc=disabled \
    -D gbm=enabled \
    -D gles1=disabled \
    -D gles2=enabled \
    -D glvnd=true \
    -D glx=dri \
    -D libunwind=disabled \
    -D llvm=enabled \
    -D lmsensors=enabled \
    -D osmesa=gallium \
    -D shared-glapi=enabled \
    -D valgrind=disabled $MESON_OPT

  # Print config
  meson configure build

  ninja -C build
  meson compile -C build

  # fake installation to be seperated into packages
  # outside of fakeroot but mesa doesn't need to chown/mod
  DESTDIR="${srcdir}/fakeinstall" meson install -C build
}

_install() {
  local src f dir
  for src; do
    f="${src#fakeinstall/}"
    dir="${pkgdir}/${f%/*}"
    install -m755 -d "${dir}"
    mv -v "${src}" "${dir}/"
  done
}

package_vulkan-mesa-layers() {
  pkgdesc="Mesa's Vulkan layers"
  depends=('libdrm' 'libxcb' 'wayland' 'python')
  conflicts=('vulkan-mesa-layer')
  replaces=('vulkan-mesa-layer')

  _install fakeinstall/usr/share/vulkan/explicit_layer.d
  _install fakeinstall/usr/lib/libVkLayer_MESA_overlay.so
  _install fakeinstall/usr/bin/mesa-overlay-control.py

  _install fakeinstall/usr/share/vulkan/implicit_layer.d
  _install fakeinstall/usr/lib/libVkLayer_MESA_device_select.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_opencl-mesa() {
  pkgdesc="OpenCL support for AMD/ATI Radeon mesa drivers"
  depends=('libdrm' 'libclc' 'clang')
  optdepends=('opencl-headers: headers necessary for OpenCL development')
  provides=('opencl-driver')

  _install fakeinstall/etc/OpenCL
  _install fakeinstall/usr/lib/lib*OpenCL*
  _install fakeinstall/usr/lib/gallium-pipe

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_vulkan-radeon() {
  pkgdesc="Radeon's Vulkan mesa driver"
  depends=('wayland' 'libx11' 'libxshmfence' 'libelf' 'libdrm' 'llvm-libs')
  optdepends=('vulkan-mesa-layers: additional vulkan layers')
  provides=('vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/radeon_icd*.json
  _install fakeinstall/usr/lib/libvulkan_radeon.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_vulkan-swrast() {
  pkgdesc="Vulkan software rasteriser driver"
  depends=('wayland' 'libx11' 'libxshmfence' 'libdrm' 'zstd' 'llvm-libs')
  optdepends=('vulkan-mesa-layers: additional vulkan layers')
  conflicts=('vulkan-mesa')
  replaces=('vulkan-mesa')
  provides=('vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/lvp_icd*.json
  _install fakeinstall/usr/lib/libvulkan_lvp.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_vulkan-broadcom() {
  pkgdesc="Broadcom's Vulkan mesa driver"
  depends=('wayland' 'libx11' 'libxshmfence' 'libdrm')
  optdepends=('vulkan-mesa-layers: additional vulkan layers')
  provides=('vulkan-driver')

  _install fakeinstall/usr/share/vulkan/icd.d/broadcom_icd*.json
  _install fakeinstall/usr/lib/libvulkan_broadcom.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_libva-mesa-driver() {
  pkgdesc="VA-API implementation for gallium"
  depends=('libdrm' 'libx11' 'llvm-libs' 'expat' 'libelf' 'libxshmfence')

  _install fakeinstall/usr/lib/dri/*_drv_video.so

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_mesa-vdpau() {
  pkgdesc="Mesa VDPAU drivers"
  depends=('libdrm' 'libx11' 'llvm-libs' 'expat' 'libelf' 'libxshmfence')

  _install fakeinstall/usr/lib/vdpau

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_mesa() {
  depends=('libdrm' 'wayland' 'libxxf86vm' 'libxdamage' 'libxshmfence' 'libelf'
           'libomxil-bellagio' 'libunwind' 'llvm-libs' 'lm_sensors' 'libglvnd'
           'zstd' 'vulkan-icd-loader')
  optdepends=('opengl-man-pages: for the OpenGL API man pages'
              'mesa-vdpau: for accelerated video playback'
              'libva-mesa-driver: for accelerated video playback')
  provides=('mesa-libgl' 'opengl-driver')
  conflicts=('mesa-libgl')
  replaces=('mesa-libgl')

  _install fakeinstall/usr/share/drirc.d/00-mesa-defaults.conf
  _install fakeinstall/usr/share/glvnd/egl_vendor.d/50_mesa.json

  # ati-dri, nouveau-dri, intel-dri, svga-dri, swrast, swr
  _install fakeinstall/usr/lib/dri/*_dri.so

  _install fakeinstall/usr/lib/bellagio
  _install fakeinstall/usr/lib/d3d
  _install fakeinstall/usr/lib/lib{gbm,glapi}.so*
  _install fakeinstall/usr/lib/libOSMesa.so*

  # in vulkan-headers
  rm -rfv fakeinstall/usr/include/vulkan

  _install fakeinstall/usr/include
  rm -f fakeinstall/usr/lib/pkgconfig/{egl,gl}.pc
  _install fakeinstall/usr/lib/pkgconfig

  # libglvnd support
  _install fakeinstall/usr/lib/libGLX_mesa.so*
  _install fakeinstall/usr/lib/libEGL_mesa.so*

  # indirect rendering
  ln -s /usr/lib/libGLX_mesa.so.0 "${pkgdir}/usr/lib/libGLX_indirect.so.0"

  # make sure there are no files left to install
  find fakeinstall -depth -print0 | xargs -0 rmdir

  install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}
