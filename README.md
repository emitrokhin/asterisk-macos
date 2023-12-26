# Build instructions for Asterisk on macOS (Intel)

Install required homebrew packages. This is my list:
- jansson
- libpq
- lua
- openssl@3
- pkg-config
- portaudio
- postgresql
- sqlite
- srtp
- unixodbc

### Create destination directory, I will use /opt/asterisk:
```bash
sudo install -o `id -un` -g admin -d /opt/asterisk
```

### Clone repo:
```bash
git clone https://github.com/emitrokhin/asterisk-macos.git
```

### Downloading and patching pjproject:
```bash
wget https://github.com/pjsip/pjproject/archive/refs/tags/2.13.1.tar.gz
tar xzf 2.13.1.tar.gz && rm 2.13.1.tar.gz

patch -p2 --forward --directory=pjproject-2.13.1 <pjproject.patch
```

### Building pjproject:
```bash
cd pjproject-2.13.1

export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig"
export CFLAGS="-I/usr/local/include -O2 -DNDEBUG"
export LDFLAGS="-L/usr/include/lib"

./configure --prefix=/opt/asterisk --enable-shared --with-ssl --disable-resample --disable-video --disable-opencore-amr --disable-speex-codec --disable-speex-aec --disable-bcg729 --disable-gsm-codec --disable-ilbc-codec --disable-l16-codec --disable-g711-codec --disable-g722-codec --disable-g7221-codec --disable-opencore-amr --disable-silk --disable-opus --disable-video --disable-v4l2 --disable-sound --disable-ext-sound --disable-sdl --disable-libyuv --disable-ffmpeg --disable-openh264 --disable-ipp --disable-libwebrtc --with-external-pa --with-external-srtp

make dep && make && make install

cd ..
```

### Downloading and patching asterisk:
```bash
wget https://github.com/asterisk/asterisk/releases/download/20.5.0/asterisk-20.5.0.tar.gz
tar xzf asterisk-20.5.0.tar.gz && rm asterisk-20.5.0.tar.gz

patch -p2 --forward --directory=asterisk-20.5.0 <asterisk.patch
```

### Building asterisk:
```bash
cd asterisk-20.5.0/

export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:/opt/asterisk/lib/pkgconfig"
export CFLAGS="-I/usr/local/include -I/opt/asterisk/include"
export LDFLAGS="-L/usr/local/lib -L/opt/asterisk/lib"

./configure --prefix=/opt/asterisk --without-pjproject-bundled --with-pjproject --without-iodbc --with-unixodbc=/usr/local/opt/unixodbc/lib --with-sqlite3=/usr/local/opt/sqlite/lib

make menuselect
make && make install
```
Optional:
```bash
make config
```
Note: During make menuselect I disable res_geolocation and res_prometheus as I had some issues with that.

### Install launchd agent (manual):
```bash
cd ..
cp asterisk.plist ~/Library/LaunchAgents

launchctl start asterisk
```
