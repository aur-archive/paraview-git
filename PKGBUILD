# Maintainer: Mathias Anselmann <mathias.anselmann@gmail.com>
# Contributor: Stéphane Gaudreault <stephane@archlinux.org>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: Michele Mocciola <mickele>
# Contributor: Simon Zilliken <simon____AT____zilliken____DOT____name>

pkgname=paraview-git
_gitname=paraview
_gitroot=git://paraview.org/ParaView.git
pkgver=4.3.1.r208.g639e265
pkgrel=1
pkgdesc='Parallel Visualization Application using VTK-Development version with manta'
arch=('i686' 'x86_64')
url='http://www.paraview.org'
license=('custom')
depends=('qtwebkit' 'openmpi' 'python2' 'ffmpeg-compat' 'boost' 'libcgns-paraview'
	 'expat' 'freetype2' 'hdf5' 'libjpeg' 'libxml2' 'libtheora' 'libpng' 'libtiff' 'zlib'
         'python2-selenium' 'python2-pillow' 'python2-matplotlib' 'python2-requests' 'python2-numpy')
makedepends=('git' 'cmake' 'mesa')
optdepends=('python2-matplotlib: Needed to support equation rendering using MathText markup language'
	    'python2-numpy: Needed for using some filters such as "Python Calculator"')
conflicts=('paraview' 'paraview-manta')
provides=('paraview' 'paraview-manta')
source=($_gitname::$_gitroot
	'paraview.png'
	'paraview.desktop'
        '0001-gcc49.patch')
sha1sums=('SKIP'
          'a2dff014e1235dfaa93cd523286f9c97601d3bbc'
          '1f94c8ff79bb2bd2c02d6b403ea1f4599616531b'
          '69e1f8c0b90ceee46feaf11c3fae7a22bf68e0dd')

pkgver() {
	cd "$srcdir/$_gitname"
	git describe --long --tags | sed -r 's/^v//;s/([^-]*-g)/r\1/;s/-/./g'
}

build() {
   cd $srcdir
  cd $_gitname
  git submodule init && git submodule update

  #Patch for mpi4py, see
  # http://www.paraview.org/pipermail/paraview/2014-February/030517.html
  patch -p1 < "${srcdir}/0001-gcc49.patch"
  
  rm -rf "${srcdir}/build"
  mkdir "${srcdir}/build"
  cd "${srcdir}/build"
  
  # flags to enable system libs
  # add PROTOBUF when http://www.vtk.org/Bug/view.php?id=13656 gets fixed
  local cmake_system_flags=""
  for lib in EXPAT FREETYPE HDF5 JPEG LIBXML2 OGGTHEORA PNG TIFF ZLIB; do
    cmake_system_flags+="-DVTK_USE_SYSTEM_${lib}:BOOL=ON "
  done

   # flags to use python2 instead of python which is 3.x.x on archlinux
   local cmake_system_python_flags="-DPYTHON_EXECUTABLE:PATH=/usr/bin/python2 \
   	 -DPYTHON_INCLUDE_DIR:PATH=/usr/include/python2.7 -DPYTHON_LIBRARY:PATH=/usr/lib/libpython2.7.so"

   # flags to use ffmpeg-compat instead of ffmpeg until
   # http://paraview.org/Bug/view.php?id=14215 gets fixed
   local ffmpeg_compat_flags="-DFFMPEG_INCLUDE_DIR:PATH=/usr/include/ffmpeg-compat \
   	 -DFFMPEG_avcodec_LIBRARY=/usr/lib/ffmpeg-compat/libavcodec.so \
   	 -DFFMPEG_avformat_LIBRARY=/usr/lib/ffmpeg-compat/libavformat.so \
   	 -DFFMPEG_avutil_LIBRARY=/usr/lib/ffmpeg-compat/libavutil.so \
   	 -DFFMPEG_swscale_LIBRARY=/usr/lib/ffmpeg-compat/libswscale.so"

   # enable when http://paraview.org/Bug/view.php?id=12852 gets fixed:
   #-DCMAKE_SKIP_RPATH:BOOL=YES \
   cmake \
   -DBUILD_SHARED_LIBS:BOOL=ON \
   -DBUILD_TESTING:BOOL=OFF \
   -DCMAKE_BUILD_TYPE=Release \
   -DCMAKE_C_COMPILER=mpicc \
   -DCMAKE_CXX_COMPILER=mpicxx \
   -DCMAKE_INSTALL_PREFIX:PATH=/usr \
   -DCMAKE_VERBOSE_MAKEFILE:BOOL=OFF \
   -DPARAVIEW_ENABLE_FFMPEG:BOOL=ON \
   -DPARAVIEW_ENABLE_PYTHON:BOOL=ON \
   -DPARAVIEW_USE_MPI:BOOL=ON \
   -DPARAVIEW_USE_VISITBRIDGE:BOOL=ON \
   -DQT_HELP_GENERATOR:FILEPATH=/usr/lib/qt4/bin/qhelpgenerator \
   -DQT_QMAKE_EXECUTABLE=qmake-qt4 \
   -DVISIT_BUILD_READER_CGNS:BOOL=ON \
   ${cmake_system_flags} \
   ${cmake_system_python_flags} \
   ${ffmpeg_compat_flags} \
   ../$_gitname

   make
}

package() {
  cd "${srcdir}/build"

  make DESTDIR="${pkgdir}" install

  #Install license
  install -Dm644 "${srcdir}/$_gitname/License_v1.2.txt" "${pkgdir}/usr/share/licenses/paraview/LICENSE"

  #Install desktop shortcuts
  install -Dm644 "${srcdir}/paraview.png" "${pkgdir}/usr/share/pixmaps/paraview.png"
  desktop-file-install --dir="${pkgdir}"/usr/share/applications "${srcdir}/paraview.desktop"
}
