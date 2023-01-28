## Play Doom on RISC-V

As D1 SOC has no 3D unit, getting modern port of Doom -- GZDoom or LZDoom requires some tinkering. Below are the steps for [Ubuntu](https://ubuntu.com/download/risc-v) to run LZDoom on [$30](https://www.aliexpress.com/item/1005004157984532.html) [RISC-V MQ-Pro board](https://mangopi.org/mqpro) in 640x480 resolution on Linux framebuffer (no Xorg) with original 8-bit software renderer at 20+fps. Truecolor and softpoly rendering works but at half the framerate. Analog audio is via the [pads](https://mangopi.org/mqpro#spectification) on the bottom side of the [board](https://linux-sunxi.org/MangoPi_MQ-Pro) or I2S audio on the pin header (RaspberryPi compatible pinout).

An alternative is to install `crispy-doom` package, but (a) you'd still need SDL2 built for KMSDRM video (below); and (b) for better framerates in [Crispy Doom](https://github.com/fabiangreffrath/crispy-doom) play with `-nosound` or recompile with FluidSynth support.

[![Crispy Doom with I2S sound via external DAC](https://img.youtube.com/vi/DicVTGy9wUw/hqdefault.jpg)](https://youtu.be/DicVTGy9wUw)]

0. **Install development packages:**
```
apt install -y \
  cmake \
  libasound2-dev \
  libdbus-1-dev \
  libdrm-dev \
  libfluidsynth-dev \
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

Someday there [will be native 2D](https://github.com/libsdl-org/SDL/issues/6570#issuecomment-1323154215) libSDL display output again.

For now, get an SDL2 [proof-of-concept](https://github.com/libsdl-org/SDL/issues/6570#issuecomment-1363905944)
built with [DRMKMS Dumb Buffers](https://github.com/JohnnyonFlame/SDL-dumbbuffers), written by João Henrique.

[SDL2.zip](https://github.com/JohnnyonFlame/SDL-dumbbuffers/archive/refs/heads/SDL2.zip)
```
cd SDL2/
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DSDL_DISKAUDIO=off \
  -DSDL_DUMMYAUDIO=off \
  -DSDL_DUMMYVIDEO=off \
  -DSDL_OFFSCREEN=off \
  -DSDL_OPENGLES=off \
  -DSDL_OSS=off \
  -DSDL_STATIC=off \
  -DSDL_VIRTUAL_JOYSTICK=off \
  -DSDL_VULKAN=off \
  -DSDL_X11=off
make && sudo make install && sudo ldconfig
```

2. **Config**

Put your `ubuntu` user in groups and re-login:
```
sudo usermod -a -G audio,video,input ubuntu
```

For I2S sound on pin header you should change Device Tree setup. Two commits of interest: [I2S](https://github.com/arkadijs/linux/commit/58919d94dc9d5f6833ee051601d17943832b5fc0)
and/or [audio pads](https://github.com/arkadijs/linux/commit/db268af4aad66f870562c3e92c02cfa0f9535298).
Or download [sun20i-d1-mangopi-mq-pro.dtb](https://github.com/arkadijs/riscv-doom/raw/main/sun20i-d1-mangopi-mq-pro.dtb) from here, then
edit `/boot/grub/grub.cfg` to set `devicetree /boot/sun20i-d1-mangopi-mq-pro.dtb`.

3. **LZDoom**

Build LZDoom as is. The build must be optimized - `Release` or `RelWithDebInfo`. Pure debug build segfaults.

[gzdoom-3.88b.tar.gz](https://github.com/drfrag666/gzdoom/archive/refs/tags/3.88b.tar.gz)
```
cd gzdoom-3.88b/
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
make
```
It will take some time.

4. **Run!**

Get some WADs, put into a folder, set `export DOOMWADDIR=$HOME/doom/wads`.
```
cd gzdoom-3.88b/build/
./lzdoom -iwad doom2.wad \
  +fullscreen false \
  +r_polyrenderer false \
  +swtruecolor false \
  +vid_defheight 480 \
  +vid_defwidth 640 \
  +vid_forcesurface true \
  +vid_renderer 0 \
  +vid_fps true \
  +snd_samplerate 48000
```
After first run put VARs into `~/.config/lzdoom/lzdoom.ini`. `snd_samplerate 48000` is for I2S sound (only), or try `-nosound`.

For I2S try any cheap [TI PCM510x](https://www.aliexpress.com/item/1005002898278583.html) or ES7148/ES7134 DAC which AliExpress is full of.

5. **Crispy Doom**

`crispy-doom` package is fine but it's missing FluidSynth support.

[master.zip](https://github.com/fabiangreffrath/crispy-doom/archive/refs/tags/master.zip)
```
cd master/
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=RelWithDebInfo
make crispy-doom
```

Run and exit to create initial config:

```
cd master/build/
./src/crispy-doom
```

Edit `~/.local/share/crispy-doom/crispy-doom.cfg`:

```
snd_samplerate     12000
snd_dmxoption      ""
fluidsynth_sf_path "/home/ubuntu/doom/sf2/gzdoom.sf2"
```

Download GZDoom SoundFont [gzdoom.sf2](https://github.com/ZDoom/gzdoom/raw/master/soundfont/gzdoom.sf2) for fast and [authentic SC-55](https://www.vogons.org/viewtopic.php?f=9&t=45600) MIDI playback.


You may also want to install [termfix](https://github.com/hobbitalastair/termfix) to repair console tty sanity if Doom fails to exit cleanly.
