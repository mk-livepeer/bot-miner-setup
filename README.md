# bot-miner-setup
setup instructions for side-by-side GPU mining and transcoding w/ LivePeer

The following installation steps assume that you are logged in as the root user.

* Start off with a clean installation of Ubuntu 18.04 LTS

* Update the current system and install software-properties-common:
```
apt-get update
apt-get dist-upgrade -y
apt-get install software-properties-common -y
```

* Install more dependencies:
```
add-apt-repository ppa:longsleep/golang-backports
apt-get update
apt-get dist-upgrade -y
apt-get install -y build-essential pkg-config autoconf gnutls-dev git curl golang-go
```

* Install CUDA drivers:
```
curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
dpkg --force-confdef --force-confnew -i ./cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
apt-get update
apt-get install cuda-10-0 -y 
```

* Install NASM:
```
git clone -b nasm-2.14.02 https://repo.or.cz/nasm.git "$HOME/nasm"
cd "$HOME/nasm"
./autogen.sh
./configure
make
make install
```
NOTE: While installing NASM, expect a failure during installation of man pages / documentation.  It should be OK otherwise - It's safe to ignore these errors.

* Install x264:
```
git clone http://git.videolan.org/git/x264.git "$HOME/x264"
cd "$HOME/x264"
git checkout 545de2ffec6ae9a80738de1b2c8cf820249a2530
./configure --enable-pic --enable-static --disable-cli
make
make install-lib-static
```

* Pull down FFMPEG source code:  We don't need this just yet, but my preference is to start pulling it down now and come back to it later.
```
git clone https://git.ffmpeg.org/ffmpeg.git "$HOME/ffmpeg"
```

* Install NV Codec Headers:
```
git clone --single-branch https://github.com/FFmpeg/nv-codec-headers
cd nv-codec-headers
make install
cd ..
rm -rf nv-codec-headers
```

Now that we've downloaded and installed most of our dependencies, it's time to build FFMPEG.  

* We will need to define the following environment:
```
export PATH=/usr/local/cuda/bin:$HOME/compiled/bin:$PATH
export PKG_CONFIG_PATH=$HOME/compiled/lib/pkgconfig
```

* Configure FFMPEG's build system:
```
cd "$HOME/ffmpeg"
git checkout 4cfc34d9a8bffe4a1dd53187a3e0f25f34023a09
./configure --disable-static --enable-shared \
        --enable-gpl --enable-nonfree --enable-libx264 --enable-cuda --enable-cuvid \
        --enable-nvenc --enable-cuda-nvcc --enable-libnpp --enable-gnutls \
        --extra-ldflags=-L/usr/local/cuda/lib64 \
        --extra-cflags='-pg -I/usr/local/cuda/include' --disable-stripping
```

* Build and install FFMPEG
```
make
make install
```

* Fetch go-livepeer using `go get`, then build livepeer and devtool:
```
go get github.com/livepeer/go-livepeer/cmd/livepeer
cd "$HOME/go/src/github.com/livepeer/go-livepeer"
go build ./cmd/livepeer/livepeer.go
go build ./cmd/devtool/devtool.go
```

* Before attempting to run the livepeer network, be sure to export a new library path:
```
ENV LD_LIBRARY_PATH /usr/local/lib/
```

TO DO:
* explain how to use devtool to set up a B/O/T network
* explain the process to download and build ethminer
* document process to setup systemd