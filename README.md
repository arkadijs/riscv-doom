## Play Doom on RISC-V

As D1 SOC has no 3D unit, getting modern port of Doom -- GZDoom or LZDoom requires some tinkering. Below are the steps for [Ubuntu](https://ubuntu.com/download/risc-v) to run LZDoom on [$30](https://www.aliexpress.com/item/1005004157984532.html) [RISC-V MQ-Pro board](https://mangopi.org/mqpro) in 640x360 resolution on Linux framebuffer (no X) with original 8-bit software renderer at 20~30fps. Truecolor and softpoly rendering works but at half the framerate. No audio so far.

Someday there [will be native 2D](https://github.com/libsdl-org/SDL/issues/6570#issuecomment-1323154215) libSDL display output again.

0. **Install development packages:**
```
apt install -y \
  libasound2-dev \
  libdbus-1-dev \
  libdrm-dev \
  libgbm-dev \
  libgl-dev \
  libgles-dev \
  libglu1-mesa-dev \
  libsamplerate0-dev \
  libudev-dev \
  zlib1g-dev \
  autoconf \
  automake \
  cmake \
  libtool \
  pkg-config
```

1. **DirectDB**

Get a working DirectFB install with minimal set of drivers. We need only software screen and no kernel _Fusion_.
[directfb_1.7.7.orig.tar.gz](http://archive.ubuntu.com/ubuntu/pool/universe/d/directfb/directfb_1.7.7.orig.tar.gz)
```
CFLAGS="-g -O2 -pipe" CXXFLAGS="-g -O2 -pipe" ./configure \
  --build=riscv64-linux-gnu \
  --disable-devmem \
  --disable-mesa \
  --disable-multi \
  --disable-multi-kernel \
  --disable-multicore \
  --disable-network \
  --disable-osx \
  --disable-sdl \
  --disable-video4linux \
  --disable-video4linux2 \
  --disable-vnc \
  --disable-x11 \
  --enable-gcc-atomics \
  --enable-zlib \
  --with-fs-drivers=none \
  --with-gfxdrivers=none \
  --with-inputdrivers=keyboard,linuxinput \
  --without-tests &&
make && sudo make install
```

2. **libSDL2**

Get an SDL2 built with DirectDB support.
[SDL-release-2.26.1.tar.gz](https://github.com/libsdl-org/SDL/archive/refs/tags/release-2.26.1.tar.gz)
```
CFLAGS="-g -O2 -pipe" CXXFLAGS="-g -O2 -pipe" ./configure \
  --disable-alsa-shared \
  --disable-arts \
  --disable-directfb-shared \
  --disable-directx \
  --disable-diskaudio \
  --disable-dummyaudio \
  --disable-esd \
  --disable-fcitx \
  --disable-fusionsound \
  --disable-ibus \
  --disable-jack \
  --disable-joystick-mfi \
  --disable-kmsdrm-shared \
  --disable-libdecor \
  --disable-nas \
  --disable-oss \
  --disable-pipewire \
  --disable-pulseaudio \
  --disable-pulseaudio-shared \
  --disable-render-d3d \
  --disable-rpath \
  --disable-sndio \
  --disable-static \
  --disable-video-cocoa \
  --disable-video-dummy \
  --disable-video-metal \
  --disable-video-opengles1 \
  --disable-video-rpi \
  --disable-video-vivante \
  --disable-video-vulkan \
  --disable-video-wayland \
  --disable-video-x11 \
  --disable-wasapi \
  --disable-wayland-shared \
  --disable-x11-shared \
  --disable-xinput \
  --enable-alsa \
  --enable-dbus \
  --enable-hidapi \
  --enable-hidapi-joystick \
  --enable-libsamplerate \
  --enable-libudev \
  --enable-sdl2-config \
  --enable-video-directfb \
  --enable-video-kmsdrm \
  --enable-video-opengl \
  --enable-video-opengles \
  --enable-video-opengles2 &&
make && sudo make install
```
Now run `ldconfig`.

3. **LZDoom**

Build LZDoom as is. The build must be optimized - `Release` or `RelWithDebInfo`. Debug build segfaults.
[gzdoom-3.88b.tar.gz](https://github.com/drfrag666/gzdoom/archive/refs/tags/3.88b.tar.gz)
```
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
make
sudo setcap cap_sys_tty_config+ep lzdoom
```
It will take some time.

4. **Config**

Put your `ubuntu` user in groups and re-login:
```
sudo usermod -a -G tty,audio,video,input ubuntu
```

Grant some TTY permissions for DirectFB (or play with `udev` rules):
```
sudo chmod g+r /dev/tty0
```

Create `~/.directfbrc`:
```
system=fbdev
mode=1920x1080
depth=32
linux-input-grab
no-banner
no-vt-switch
vt-switching
vsync-none
```

5. **Run!**

Get some WADs, put into a folder, set `export DOOMWADDIR=$HOME/wads`.
```
cd gzdoom-3.88b/build/
./lzdoom -iwad doom2.wad \
  +fullscreen false \
  +r_polyrenderer false \
  +swtruecolor false \
  +vid_defheight 360 \
  +vid_defwidth 640 \
  +vid_forcesurface true \
  +vid_renderer 0 \
  +vid_fps true
```
After first run put VARs into `~/.config/lzdoom/lzdoom.ini`.

You may also want to install [termfix](https://github.com/hobbitalastair/termfix) to repair console tty sanity if Doom / DirectFB fails to exit cleanly.
