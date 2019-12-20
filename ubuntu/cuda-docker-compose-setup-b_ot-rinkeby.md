***CUDA GPU-ENABLED DOCKER-COMPOSE B/OT-NETWORK***

* Log in as a non-root user with sudo rights, then execute the following block of commands to install `nvidia-docker2`:

```bash
export distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
export DEBIAN_FRONTEND=noninteractive
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
curl -O http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.1.243-1_amd64.deb
sudo dpkg --force-confdef --force-confnew -i ./cuda-repo-ubuntu1804_10.1.243-1_amd64.deb
sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
sudo apt-get update
sudo apt-get install -y cuda-drivers nvidia-docker2
```

Confirm that the drivers were installed properly using the `nvidia-smi` utility:

```bash
nvidia-smi
```

If `nvidia-smi` reports a driver version mismatch error, you will have to reboot the machine:

```bash
sudo reboot
```

* Update `/etc/docker/daemon.json` to add "nvidia-container-runtime" and make it the default runtime.

On a clean install, you will have to edit the file and add a comma `,` followed by the `"default-runtime": "nvidia"` line:

```bash
sudo nano /etc/docker/daemon.json
```

The end result should resemble the following:

```json
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "default-runtime": "nvidia"
}
```

Hit `CTRL-X` to save and exit.  When prompted, hit the `Y` key followed by `ENTER`.

* Restart docker:

```bash
sudo systemctl restart docker
```

* Install docker-compose:

```bash
sudo apt-get install docker-compose
```

* Use the following command:

```bash
nano docker-compose.yml
```

...to create a file called `docker-compose.yml` with the following contents:

```yaml
version: '3.5'
services:
  orchtran:
    image: livepeer/go-livepeer:master
    command: '-network rinkeby -orchestrator -transcoder -serviceAddr orchtran:8935 -orchAddr 0.0.0.0 -initializeRound=true -pricePerUnit 1 -nvidia 0'
    ports:
      - 7935:7935
      - 8935:8935
    volumes:
      - orchroot:/root
  broadcaster:
    depends_on:
      - orchtran
    image: livepeer/go-livepeer:master
    command: '-broadcaster -network rinkeby -rtmpAddr broadcaster -orchAddr orchtran:8935 -cliAddr broadcaster:7936 -httpAddr broadcaster:8936 -depositMultiplier 1'
    ports:
      - 1935:1935
      - 7936:7936
      - 8936:8936
    volumes:
      - bcstroot:/root
volumes:
  orchroot:
  bcstroot:
```

If you have multiple GPU's, adjust the `-nvidia` option on the transcoder container's command line to account for them.
For example, for two Nvidia GPUs, specify `-nvidia 0,1`
If you don't have any Nvidia GPUs, remove the `-nvidia 0` option from the transcoder container command line.

Hit `CTRL-X` to save and exit.  When prompted, hit the `Y` key followed by `ENTER`.

* Before starting the B/OT network, we must initiatize an ethereum account.  Begin by bringing up the orchestrator/transcoder alone using interactive mode:

```bash
sudo docker-compose run orchtran
```

go-livepeer will prompt for a passphrase.  Enter one and then press `ENTER` to continue.  It will ask you to repeat the passphrase, then enter it again to unlock the new ethereum account.

After this is done, go-livepeer may shut down.  Feel free to stop it using `CTRL-C` if it doesnt shut down by itself after some time.

* Store your password in a file on disk in a secure location.

```bash
echo MyEthPassPhrase > passphrase_orch.txt
```

* Mount the file within the container by adding " - ./passphrase_orch.txt:/root/pw.txt" to the volumes section of the orchtran container in `docker-compose.yml`.  It should look like this:

```yaml
    volumes:
      - orchroot:/root
      - ./passphrase_orch.txt:/root/pw.txt
```

* Edit the `docker-compose.yml` to add the `ethPassword` argument to the orchtran command line.  The new command should look like this:

```bash
-network rinkeby -orchestrator -transcoder -serviceAddr orchestrator:8935 -orchAddr 0.0.0.0 -initializeRound=true -pricePerUnit 1 -nvidia 0 -ethPassword=/root/pw.txt
```

* Next, we must initialize an ethereum account for the broadcaster.  Bring up the broadcaster using interactive mode:

```bash
sudo docker-compose run broadcaster
```

go-livepeer will prompt for a passphrase.  Enter one and then press `ENTER` to continue.  It will ask you to repeat the passphrase, then enter it again to unlock the new ethereum account.

After this is done, go-livepeer may shut down.  Feel free to stop it using `CTRL-C` if it doesnt shut down by itself after some time.

* Store your password in a file on disk in a secure location.

```bash
echo MyEthPassPhrase > passphrase_bcst.txt
```

* Mount the file within the container by adding " - ./passphrase_bcst.txt:/root/pw.txt" to the volumes section of the broadcaster container in `docker-compose.yml`.  It should look like this:

```yaml
    volumes:
      - bcstroot:/root
      - ./passphrase_bcst.txt:/root/pw.txt
```

* Edit the `docker-compose.yml` to add the `ethPassword` argument to the broadcaster command line.  The new command should look like this:

```bash
-broadcaster -network rinkeby -rtmpAddr broadcaster -orchAddr orchestrator:8935 -cliAddr broadcaster:7936 -httpAddr broadcaster:8936 -depositMultiplier 1 -ethPassword=/root/pw.txt
```

Now, we must fund the accounts with ETH and LPT.  We'll have to bring up the B/O/T network and then execute the CLI in another shell.

* Start the B/O/T network using the following command:

```bash
sudo docker-compose up
```

* Using another shell, list the running containers to see their names:

```bash
docker container ls
```

* Start the orchtran CLI with the following command (replace "orchtran" with the actual container name as seen above):

```bash
docker container exec -it orchtran livepeer_cli
```

* Follow the instructions in the CLI to first get some ETH, then some LPT.  After both are done, you may exit the CLI.

* Start the broadcaster CLI with the following command (replace "broadcaster" with the actual container name as seen above):

```bash
docker container exec -it broadcaster livepeer_cli --http 7936 --host broadcaster
```

* Follow the instructions in the CLI to first get some ETH, then some LPT.  After both are done, you may exit the CLI.

* You may leave the B/O/T network running if you like, but if you want to stop the B/O/T network, use the following command:

```bash
sudo docker-compose down
```

Now the system is ready to run!

* Start the B/OT network using the following command:

```bash
sudo docker-compose up
```

You should see output similar to:

```Recreating michael_orchtran_1 ... 
Recreating michael_orchtran_1 ... done
Recreating michael_broadcaster_1 ... 
Recreating michael_broadcaster_1 ... done
Attaching to michael_orchtran_1, michael_broadcaster_1
orchtran_1     | I1210 18:27:15.858357       1 livepeer.go:199] ***Livepeer is running on the rinkeby network: 0xA268AEa9D048F8d3A592dD7f1821297972D4C8Ea***
orchtran_1     | I1210 18:27:16.175813       1 accountmanager.go:70] Using Ethereum account: 0x02452e64DA3959f7b3e487e27a8CD672808e49C2
broadcaster_1  | I1210 18:27:17.321270       1 livepeer.go:199] ***Livepeer is running on the rinkeby network: 0xA268AEa9D048F8d3A592dD7f1821297972D4C8Ea***
broadcaster_1  | I1210 18:27:17.576022       1 accountmanager.go:70] Using Ethereum account: 0x7fe2e5bb9Ea5533c68A011466880FB6Bbaa2115A
orchtran_1     | I1210 18:27:17.673865       1 accountmanager.go:99] Unlocked ETH account: 0x02452e64DA3959f7b3e487e27a8CD672808e49C2
orchtran_1     | I1210 18:27:18.091648       1 livepeer.go:463] Price: 1 wei for 1 pixels
orchtran_1     |  
orchtran_1     | I1210 18:27:18.149597       1 livepeer.go:860] Orchestrator 0x02452e64DA3959f7b3e487e27a8CD672808e49C2 is inactive
orchtran_1     | I1210 18:27:18.234251       1 block_watcher.go:318] Backfilling block events (this can take a while)...
orchtran_1     | I1210 18:27:18.234840       1 block_watcher.go:319] Start block: 5590875                End block: 5590878             Blocks elapsed: 3
orchtran_1     | I1210 18:27:18.251527       1 block_watcher.go:537] fetching block logs from=5590875 to=5590878
orchtran_1     | I1210 18:27:18.382191       1 livepeer.go:740] ***Livepeer Running in Orchestrator Mode***
orchtran_1     | I1210 18:27:18.382360       1 webserver.go:68] CLI server listening on 127.0.0.1:7935
orchtran_1     | I1210 18:27:18.382684       1 cert.go:83] Private key and cert not found. Generating
orchtran_1     | I1210 18:27:18.382970       1 cert.go:22] Generating cert for orchtran
orchtran_1     | I1210 18:27:18.383456       1 rpc.go:147] Listening for RPC on :8935
broadcaster_1  | I1210 18:27:18.623740       1 accountmanager.go:99] Unlocked ETH account: 0x7fe2e5bb9Ea5533c68A011466880FB6Bbaa2115A
broadcaster_1  | I1210 18:27:19.294466       1 livepeer.go:563] Maximum transcoding price per pixel is not greater than 0: 0, broadcaster is currently set to accept ANY price.
broadcaster_1  | I1210 18:27:19.294542       1 livepeer.go:564] To update the broadcaster's maximum acceptable transcoding price per pixel, use the CLI or restart the broadcaster with the appropriate 'maxPricePerUnit' and 'pixelsPerUnit' values
broadcaster_1  | I1210 18:27:19.338526       1 block_watcher.go:318] Backfilling block events (this can take a while)...
broadcaster_1  | I1210 18:27:19.338552       1 block_watcher.go:319] Start block: 5590875                End block: 5590878             Blocks elapsed: 3
broadcaster_1  | I1210 18:27:19.342778       1 block_watcher.go:537] fetching block logs from=5590875 to=5590878
broadcaster_1  | I1210 18:27:19.460729       1 livepeer.go:742] ***Livepeer Running in Broadcaster Mode***
broadcaster_1  | I1210 18:27:19.460782       1 livepeer.go:743] Video Ingest Endpoint - rtmp://broadcaster:1935
broadcaster_1  | I1210 18:27:19.460908       1 webserver.go:68] CLI server listening on broadcaster:7936
orchtran_1     | I1210 18:27:20.383165       1 rpc.go:215] Connecting RPC to https://orchtran:8935
orchtran_1     | I1210 18:27:20.387275       1 rpc.go:187] Received Ping request
```

Leave this running.

* You will have to acquire testnet eth and lpt before proceeding.  That process is outside the scope of this document.  See [livepeer.readthedocs.io](https://livepeer.readthedocs.io/en/latest/streamflow-public-testnet.html)

* In another terminal, stream into port 1935 using `ffmpeg`

```bash
ffmpeg -re -stream_loop -1 -i SomeVideo.mp4 -c:a copy -c:v copy -f flv rtmp://localhost:1935/movie
```

* In yet another terminal, playback from port 8936 using `ffplay`

```bash
ffplay http://localhost:8936/stream/movie.m3u8
```

To run ethminer side by side with the livepeer b/o/t network, complete the following additional steps:

* [install cuda drivers](../../install-cuda.md)

* [ethminer systemd setup](../../ethminer-systemd-setup.md)
