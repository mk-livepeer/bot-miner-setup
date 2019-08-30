# bot-miner-setup

setup instructions for side-by-side GPU mining and transcoding w/ LivePeer

As time goes on, alternative installation options will be added, but for now, this document is based on Ubuntu LTS.

The following installation steps assume that you are logged in as the root user.

***GO-LIVEPEER***

* Start off with a clean installation of Ubuntu 18.04 LTS.  This should also work with Ubuntu 16.04, but 18.04 LTS is recommended.

* Log in as the root user.  If you usually use `sudo` you can escalate by running:

```bash
sudo su
```

* Update the current system and install software-properties-common:

```bash
apt-get update
apt-get dist-upgrade -y
apt-get install software-properties-common -y
```

* Install dependencies:

```bash
add-apt-repository ppa:longsleep/golang-backports
```

Press "Y".  Then continue:

```
apt-get update
apt-get dist-upgrade -y
apt-get install -y build-essential pkg-config autoconf gnutls-dev git curl golang-go
```

* Install CUDA drivers:

```bash
curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
dpkg --force-confdef --force-confnew -i ./cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
apt-get update
apt-get install cuda-10-0 -y 
```

* Install NASM:

```bash
git clone -b nasm-2.14.02 https://repo.or.cz/nasm.git "$HOME/nasm"
cd "$HOME/nasm"
./autogen.sh
./configure
make
make install
```

NOTE: While installing NASM, expect a failure during installation of man pages / documentation.  It should be OK otherwise - It's safe to ignore these errors.

* Install x264:

```bash
git clone http://git.videolan.org/git/x264.git "$HOME/x264"
cd "$HOME/x264"
git checkout 545de2ffec6ae9a80738de1b2c8cf820249a2530
./configure --enable-pic --enable-static --disable-cli
make
make install-lib-static
```

* Install NV Codec Headers:

```bash
git clone --single-branch https://github.com/FFmpeg/nv-codec-headers
cd nv-codec-headers
make install
cd ..
rm -rf nv-codec-headers
```

Now that we've downloaded and installed most of our dependencies, it's time to build FFMPEG.  

* We will need to define the following environment:

```bash
export PATH=/usr/local/cuda/bin:$HOME/compiled/bin:$PATH
export PKG_CONFIG_PATH=$HOME/compiled/lib/pkgconfig
```

* Pull down FFMPEG source code:

```bash
git clone https://git.ffmpeg.org/ffmpeg.git "$HOME/ffmpeg"
```

* Configure FFMPEG's build system:

```bash
cd "$HOME/ffmpeg"
git checkout 4cfc34d9a8bffe4a1dd53187a3e0f25f34023a09
./configure --disable-static --enable-shared \
        --enable-gpl --enable-nonfree --enable-libx264 --enable-cuda --enable-cuvid \
        --enable-nvenc --enable-cuda-nvcc --enable-libnpp --enable-gnutls \
        --extra-ldflags=-L/usr/local/cuda/lib64 \
        --extra-cflags='-pg -I/usr/local/cuda/include' --disable-stripping
```

* Build and install FFMPEG:

```bash
make
make install
```

* Add a livepeer user:

```bash
adduser --disabled-password livepeer
```

* Impersonate the livepeer user:

```bash
su - livepeer
```

* Fetch go-livepeer using `go get`, then build livepeer and devtool:

```bash
go get github.com/livepeer/go-livepeer/cmd/livepeer
cd "$HOME/go/src/github.com/livepeer/go-livepeer"
go build ./cmd/livepeer/livepeer.go
go build ./cmd/devtool/devtool.go
```

* Go-livepeer should be ready to go!  Exit the shell to return control back to the root user:

```bash
exit
```

***ETHMINER***

We're going to run the open-source `ethminer` as a systemd service.

* We need to install another dependency:

```bash
apt-get install -y cmake libdbus-1-dev
```

* Create a new user account:

```bash
adduser --disabled-password ethminer
```

* Impersonate the new user:

```bash
su - ethminer
```

* Clone the `ethereum-mining/ethminer` source code repository:

```bash
git clone https://github.com/ethereum-mining/ethminer.git
```

* Enter the directory and update submodules:

```bash
cd ethminer
git submodule update --init --recursive
```

* Create a build directory:

```bash
mkdir build
cd build
```

* Configure the project with CMake.  Additional options are defined in the [ethminer docs](https://github.com/ethereum-mining/ethminer/blob/master/docs/BUILD.md#cmake-configuration-options):

```bash
cmake ..
```

* Build the project:

```bash
make
```

* Log out of the `ethminer` user account back to the root account:

```bash
exit
```

* Now, create the `ethminer.service` file:

```bash
vi /etc/systemd/system/ethminer.service
```

* Push the `i` key to get into insertion mode.

* Paste the following:

```
[Unit]
Description=Ethereum miner daemon service
After=network.target

[Service]
User=ethminer
Group=ethminer
Type=simple
Restart=always
RestartSec=90s
WorkingDirectory=/home/ethminer/ethminer/build/
ExecStart=/home/ethminer/ethminer/build/ethminer/ethminer -U -P stratum1+tcp://0x9f3aca1541c59179269035140da607bb7b40c3eb.MINERID@us.eth.wattpool.net:8008

[Install]
WantedBy=default.target
```

Replace `MINERID` with a unique name, to identify this miner from others.  Also, feel free to replace the Ethereum address with your own.  The address above is being used by LivePeer for development and experimentation.

* Enable and start the service:

```bash
systemctl enable ethminer.service
systemctl start ethminer.service
```

* To monitor the service:

```bash
journalctl -f --unit=ethminer.service
```

TO DO:
* Before attempting to run the livepeer network, be sure to export a new library path:
```
export LD_LIBRARY_PATH=/usr/local/lib/
```

* explain how to setup a B/O/T network in OFFLINE mode

-- or --

* Download and install docker, using the instructions, here:  {docker install instructions link}

* docker run darkdragon.geth bla b lah blah

* explain how to use devtool to set up a B/O/T network on the livepeer testnet

* run devtool orchestrator/transcoder

* run devtool broadcaster

--

* stream into port 1935 w/ ffmpeg

* playback from port 8945 w/ ffplay