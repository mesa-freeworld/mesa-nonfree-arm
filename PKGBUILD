# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Maintainer: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>

# ALARM: Kevin Mihelich <kevin@archlinuxarm.org>
#  - Removed DRI and Gallium3D drivers/packages for chipsets that don't exist in our ARM devices (intel, VMware svga).
#  - disable assembly and rip out VC4 forced NEON for v6/v7
#  - remove makedepend on valgrind, -Dvalgrind=false

pkgbase=mesa
pkgname=('vulkan-mesa-layers' 'opencl-mesa' 'vulkan-radeon' 'libva-mesa-driver' 'mesa-vdpau' 'mesa')
pkgdesc="An open-source implementation of the OpenGL specification"
pkgver=20.2.6
pkgrel=0.2
arch=('x86_64' 'aarch64')
makedepends=('python-mako' 'libxml2' 'libx11' 'xorgproto' 'libdrm' 'libxshmfence' 'libxxf86vm'
             'libxdamage' 'libvdpau' 'libva' 'wayland' 'wayland-protocols' 'zstd' 'elfutils' 'llvm'
             'libomxil-bellagio' 'libclc' 'clang' 'libglvnd' 'libunwind' 'lm_sensors' 'libxrandr'
             'glslang' 'vulkan-icd-loader' 'meson')
url="https://www.mesa3d.org/"
license=('custom')
source=(https://mesa.freedesktop.org/archive/mesa-${pkgver}.tar.xz
        0001-radeonsi-fix-regression-on-gpus-using-the-radeon-winsys.patch
        0001-Rip-out-VC4-forced-NEON.patch
        LICENSE)
sha512sums=('347b275d88c0d14cacef570ed736cac07f2e607bc4c89a16b915ec01ac57dbbe698ddf9a0ad70f034e00318403351e3c728e74c72c653acf1fc99720887fa888'
            'b9c3a8d6a75df280510821db6593899e528fcf161751dec31c626893b0e817b1b6720bc1b360278defd357be3e12913188f89353059040a43acf449c1ab9a0b5'
            '626c493d109a7f6448848f611480b19bba174e2fab14707f196c5df02b124b6e65d982f93e81c20e0e711da9779ace8a0cb527822aae29411dc139f8919b36b2'
            'ff1be960d03d7793b493f5ab79ae83d6ec9c4b335de96c2f72ff358189dc40977e31b46f9add7471c5ecb6f7875fa6f1472f4b26b1ac38794455a6a03aa2b667')

prepare() {
  cd mesa-$pkgver

  patch -Np1 < ../0001-radeonsi-fix-regression-on-gpus-using-the-radeon-winsys.patch

  [[ $CARCH == "armv6h" || $CARCH == "armv7h" ]] && patch -p1 -i ../0001-Rip-out-VC4-forced-NEON.patch || true
}

build() {
  MESON_OPT="-D asm=false"
  case "${CARCH}" in
    armv6h)  GALLIUM=",vc4" ;;
    armv7h)  GALLIUM=",etnaviv,kmsro,lima,panfrost,tegra,v3d,vc4" ;;
    aarch64) GALLIUM=",kmsro,lima,panfrost,v3d,vc4" ;;
  esac

  arch-meson mesa-$pkgver build \
    -D b_lto=false \
    -D b_ndebug=true \
    -D platforms=x11,wayland \
    -D dri-drivers=r100,r200,nouveau \
    -D gallium-drivers=r300,r600,radeonsi,freedreno,nouveau,swrast,virgl,zink${GALLIUM} \
    -D vulkan-drivers=amd \
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
