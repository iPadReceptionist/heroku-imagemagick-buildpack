FROM heroku/heroku:24-build

# heroku-24 build images default to the non-root `heroku` user; switch to root so apt-get
# can install build dependencies. (heroku-22 ran as root by default.)
USER root

ENV DEBIAN_FRONTEND=noninteractive

ARG COMPILE_VERSION_LIBDE265=1.0.9
ARG COMPILE_VERSION_LIBAOM=3.5.0
# libheif 1.14.0 fails on gcc 13 (Ubuntu 24.04) with -Werror=maybe-uninitialized in
# heif_colorconversion.cc. 1.15.x has the fix.
ARG COMPILE_VERSION_LIBHEIF=1.15.2
# ImageMagick 7.1.0-53 pre-dates gcc 13; bumping to the 7.1.1 line for the same reason.
ARG COMPILE_VERSION_LIBWEBP=1.2.4
ARG COMPILE_VERSION_IMAGEMAGICK=7.1.1-38

RUN apt-get -yq update \
  # yasm for libaom cmake optimizations; lib*-dev headers so ImageMagick's configure links
  # against the stack's system JPEG/PNG/TIFF/GIF libraries on Ubuntu 24.04. Pinning these
  # explicitly guarantees reproducibility if the base image ever drops pre-installed headers.
  && apt-get install -yq yasm libtiff-dev libjpeg-dev libpng-dev libgif-dev \
  # Install the latest libde265 library
  && curl -sSL --retry 3 https://github.com/strukturag/libde265/releases/download/v$COMPILE_VERSION_LIBDE265/libde265-$COMPILE_VERSION_LIBDE265.tar.gz | tar zx \
  && cd libde265-$COMPILE_VERSION_LIBDE265 \
  && ./autogen.sh \
  && ./configure \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd .. \
  # Install the latest libwebp library
  && curl -sSL --retry 3 http://storage.googleapis.com/downloads.webmproject.org/releases/webp/libwebp-$COMPILE_VERSION_LIBWEBP.tar.gz | tar zx \
  && cd libwebp-$COMPILE_VERSION_LIBWEBP \
  && ./autogen.sh \
  && ./configure --enable-shared --enable-libwebpdecoder --enable-libwebpdemux --enable-libwebpmux --enable-static=no \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd .. \
  # Install the latest libaom library
  && git clone -b v${COMPILE_VERSION_LIBAOM} --depth 1 https://aomedia.googlesource.com/aom \
  && mkdir build_aom \
  && cd build_aom \
  && cmake ../aom/ -DENABLE_TESTS=0 -DBUILD_SHARED_LIBS=1 \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd .. \
  # Install the latest libheif library
  && curl -sSL --retry 3 https://github.com/strukturag/libheif/releases/download/v$COMPILE_VERSION_LIBHEIF/libheif-$COMPILE_VERSION_LIBHEIF.tar.gz | tar zx \
  && cd libheif-$COMPILE_VERSION_LIBHEIF \
  && ./autogen.sh \
  && ./configure \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd .. \
  # Get, configure and install ImageMagick
  && curl -sSL --retry 3 https://github.com/ImageMagick/ImageMagick/archive/refs/tags/$COMPILE_VERSION_IMAGEMAGICK.tar.gz | tar zx \
  && cd ImageMagick-$COMPILE_VERSION_IMAGEMAGICK \
  && ./configure --prefix=/usr/src/imagemagick --disable-docs --with-avif=yes --with-heic=yes --with-webp=yes --with-tiff=yes --with-openexr=no --with-x=no \
  && make -j$(nproc) \
  && make install \
  && ldconfig /usr/local/lib \
  && cd .. \
  # Copy shared libraries just compiled
  && cp /usr/local/lib/libaom.so.3 /usr/src/imagemagick/lib \
  && cp /usr/local/lib/libde265.so.0 /usr/src/imagemagick/lib \
  && cp /usr/local/lib/libheif.so.1 /usr/src/imagemagick/lib \
  && cp /usr/local/lib/libwebp.so.7 /usr/src/imagemagick/lib \
  # Clean up the build and get ready for packaging
  && cd /usr/src/imagemagick \
  && strip lib/*.a lib/lib*.so* \
  # Wrap it up with a bow (compress the binary)
  && rm -rf build \
  && mkdir build \
  && tar cjf /usr/src/imagemagick/build/imagemagick.tar.bz2 bin include lib etc share
