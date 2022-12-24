## Play Doom on RISC-V

As D1 SOC has no 3D unit, getting modern port of Doom -- GZDoom or LZDoom requires some tinkering. Below are the steps for [Ubuntu](https://ubuntu.com/download/risc-v) to run LZDoom on [$30](https://www.aliexpress.com/item/1005004157984532.html) [RISC-V MQ-Pro board](https://mangopi.org/mqpro) in 640x360 resolution on Linux framebuffer (no X) with original 8-bit software renderer at 20~30fps. Truecolor and softpoly rendering works but at half the framerate. No audio so far.

Someday there [will be native 2D](https://github.com/libsdl-org/SDL/issues/6570#issuecomment-1323154215) libSDL display output again.

0. **Install development packages:**
```
apt install -y \
  cmake \
  libasound2-dev \
  libdbus-1-dev \
  libdrm-dev \
  libgbm-dev \
  libgl-dev \
  libgles-dev \
  libglu1-mesa-dev \
  libopenal-dev \
  libsamplerate0-dev \
  libtool \
  libudev-dev \
  pkg-config \
  unzip \
  zlib1g-dev
```

1. **libSDL2**

Get an SDL2 [proof-of-concept](https://github.com/libsdl-org/SDL/issues/6570#issuecomment-1363905944)
built with [DRMKMS Dumb Buffers](https://github.com/JohnnyonFlame/SDL-dumbbuffers), written by João Henrique.

[SDL2.zip](https://github.com/JohnnyonFlame/SDL-dumbbuffers/archive/refs/heads/SDL2.zip)
```
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DSDL_STATIC=off \
  -DSDL_X11=off \
  -DSDL_VULKAN=off \
  -DSDL_OFFSCREEN=off \
  -DSDL_DISKAUDIO=off \
  -DSDL_DUMMYAUDIO=off \
  -DSDL_DUMMYVIDEO=off \
  -DSDL_OFFSCREEN=off \
  -DSDL_OSS=off \
  -DSDL_VIRTUAL_JOYSTICK=off \
  -DSDL_OPENGLES=off &&
make && sudo make install && sudo ldconfig
```

2. **LZDoom**

Build LZDoom as is. The build must be optimized - `Release` or `RelWithDebInfo`. Pure debug build segfaults.

[gzdoom-3.88b.tar.gz](https://github.com/drfrag666/gzdoom/archive/refs/tags/3.88b.tar.gz)
```
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
make
```
It will take some time.

3. **Config**

Put your `ubuntu` user in groups and re-login:
```
sudo usermod -a -G audio,video,input ubuntu
```

4. **Run!**

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

You may also want to install [termfix](https://github.com/hobbitalastair/termfix) to repair console tty sanity if Doom fails to exit cleanly.
