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

Either fill in the user details, or press `<ENTER>` a few times to accept the defaults.

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

* Save the document and exit `vi` by first hitting the `ESC` key, then `:` followed by `wq` and `<ENTER>`

* Enable and start the service:

```bash
systemctl enable ethminer.service
systemctl start ethminer.service
```

* To monitor the service:

```bash
journalctl -f --unit=ethminer.service
```

* To monitor the mining pool-side, navigate to:

https://ethereum.wattpool.net/account/0x9f3aca1541c59179269035140da607bb7b40c3eb

...and scroll down to view your miner by name.

`mk-sidebyside-0` is the name of the instance I am using to verify these instructions.

`reepevil` (livepeer spelled backwards) is the name of the rig running in the NYC office.

NOTE:  Pool-side effective hashrates are based on a comparitive analysis of submitted shares in tandem with network difficulty.  The hashrate number that matters to us most is the figure reported locally by the miner, itself.
