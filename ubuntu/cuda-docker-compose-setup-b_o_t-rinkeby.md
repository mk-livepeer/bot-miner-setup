***CUDA GPU-ENABLED DOCKER-COMPOSE B/O/T-NETWORK***

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
  orchestrator:
    image: livepeer/go-livepeer:master
    command: '-orchestrator -network rinkeby -orchSecret /secret.txt -serviceAddr orchestrator:8935 -orchAddr 0.0.0.0 -pricePerUnit 1 -initializeRound=true'
    ports:
      - 7935:7935
      - 8935:8935
    volumes:
      - orchroot:/root
      - osecret.txt:/secret.txt
  transcoder:
    depends_on:
      - orchestrator
    image: livepeer/go-livepeer:master
    command: '-transcoder -network rinkeby -orchAddr orchestrator:8935 -orchSecret /secret.txt -nvidia 0'
    volumes:
      - osecret.txt:/secret.txt
  broadcaster:
    depends_on:
      - orchestrator
      - transcoder
    image: livepeer/go-livepeer:master
    command: '-broadcaster -network rinkeby -rtmpAddr broadcaster -orchAddr orchestrator:8935 -cliAddr broadcaster:7936 -httpAddr broadcaster:8936 -depositMultiplier 1'
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

* Before starting the B/O/T network, we must initiatize an ethereum account.  Begin by bringing up the orchestrator alone using interactive mode:

```bash
sudo docker-compose run orchestrator
```

go-livepeer will prompt for a passphrase.  Enter one and then press `ENTER` to continue.  It will ask you to repeat the passphrase, then enter it again to unlock the new ethereum account.

After this is done, go-livepeer may shut down.  Feel free to stop it using `CTRL-C` if it doesnt shut down by itself after some time.

* Store your orchSecret password in a text file. Replace "secret" with a secret of your own:

```bash
echo secret > osecret.txt
```

* Store your password in a file on disk in a secure location.

```bash
echo MyEthPassPhrase > passphrase_orch.txt
```

* Mount the file within the container by adding " - ./passphrase_orch.txt:/root/pw.txt" to the volumes section of the orchestrator container in `docker-compose.yml`.  It should look like this:

```yaml
    volumes:
      - orchroot:/root
      - osecret.txt:/secret.txt
      - ./passphrase_orch.txt:/root/pw.txt
```

* Edit the `docker-compose.yml` to add the `ethPassword` argument to the orchestrator command line.  The new command should look like this:

```bash
-orchestrator -network rinkeby -orchSecret /secret.txt -serviceAddr orchestrator:8935 -orchAddr 0.0.0.0 -pricePerUnit 1 -initializeRound=true -ethPassword=/root/pw.txt
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

* Start the orchestrator CLI with the following command (replace "orchestrator" with the actual container name as seen above):

```bash
docker container exec -it orchestrator livepeer_cli
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

* Start the B/O/T network using the following command:

```bash
sudo docker-compose up
```

You should see output similar to:

```
Creating network "michael_default" with the default driver
Creating michael_orchestrator_1 ... 
Creating michael_orchestrator_1 ... done
Creating michael_transcoder_1 ... 
Creating michael_transcoder_1 ... done
Creating michael_broadcaster_1 ... 
Creating michael_broadcaster_1 ... done
Attaching to michael_orchestrator_1, michael_transcoder_1, michael_broadcaster_1
orchestrator_1  | I1210 17:20:22.018354       1 livepeer.go:199] ***Livepeer is running on the rinkeby network: 0xA268AEa9D048F8d3A592dD7f1821297972D4C8Ea***
transcoder_1    | I1210 17:20:23.307593       1 livepeer.go:199] ***Livepeer is running on the rinkeby network: 0xA268AEa9D048F8d3A592dD7f1821297972D4C8Ea***
orchestrator_1  | I1210 17:20:22.401746       1 accountmanager.go:70] Using Ethereum account: 0x02452e64DA3959f7b3e487e27a8CD672808e49C2
orchestrator_1  | I1210 17:20:24.021618       1 accountmanager.go:99] Unlocked ETH account: 0x02452e64DA3959f7b3e487e27a8CD672808e49C2
transcoder_1    | I1210 17:20:23.307915       1 livepeer.go:214] Creating data dir: /root/.lpData/rinkeby
broadcaster_1   | I1210 17:20:24.497440       1 livepeer.go:199] ***Livepeer is running on the rinkeby network: 0xA268AEa9D048F8d3A592dD7f1821297972D4C8Ea***
orchestrator_1  | I1210 17:20:24.507397       1 livepeer.go:463] Price: 1 wei for 1 pixels
orchestrator_1  |  
transcoder_1    | I1210 17:20:23.406314       1 livepeer.go:277] ***Livepeer is in transcoder mode ***
transcoder_1    | I1210 17:20:23.406378       1 ot_rpc.go:50] Registering transcoder to orchestrator:8935
transcoder_1    | E1210 17:20:23.412110       1 ot_rpc.go:96] Could not register transcoder to orchestrator rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 172.21.0.2:8935: connect: connection refused"
transcoder_1    | I1210 17:20:23.412190       1 ot_rpc.go:52] Unregistering transcoder: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 172.21.0.2:8935: connect: connection refused"
transcoder_1    | I1210 17:20:23.960117       1 ot_rpc.go:50] Registering transcoder to orchestrator:8935
transcoder_1    | E1210 17:20:23.961343       1 ot_rpc.go:96] Could not register transcoder to orchestrator rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 172.21.0.2:8935: connect: connection refused"
transcoder_1    | I1210 17:20:23.961366       1 ot_rpc.go:52] Unregistering transcoder: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 172.21.0.2:8935: connect: connection refused"
orchestrator_1  | I1210 17:20:24.560797       1 livepeer.go:860] Orchestrator 0x02452e64DA3959f7b3e487e27a8CD672808e49C2 is inactive
transcoder_1    | I1210 17:20:24.617384       1 ot_rpc.go:50] Registering transcoder to orchestrator:8935
transcoder_1    | E1210 17:20:24.619310       1 ot_rpc.go:96] Could not register transcoder to orchestrator rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 172.21.0.2:8935: connect: connection refused"
transcoder_1    | I1210 17:20:24.619745       1 ot_rpc.go:52] Unregistering transcoder: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 172.21.0.2:8935: connect: connection refused"
orchestrator_1  | I1210 17:20:24.671523       1 block_watcher.go:318] Backfilling block events (this can take a while)...
orchestrator_1  | I1210 17:20:24.671549       1 block_watcher.go:319] Start block: 5590609               End block: 5590610             Blocks elapsed: 1
orchestrator_1  | I1210 17:20:24.675363       1 block_watcher.go:537] fetching block logs from=5590609 to=5590610
broadcaster_1   | I1210 17:20:24.703313       1 accountmanager.go:70] Using Ethereum account: 0x02452e64DA3959f7b3e487e27a8CD672808e49C2
orchestrator_1  | I1210 17:20:24.869675       1 livepeer.go:740] ***Livepeer Running in Orchestrator Mode***
orchestrator_1  | I1210 17:20:24.870334       1 webserver.go:68] CLI server listening on 127.0.0.1:7935
orchestrator_1  | I1210 17:20:24.870467       1 cert.go:83] Private key and cert not found. Generating
orchestrator_1  | I1210 17:20:24.871715       1 cert.go:22] Generating cert for orchestrator
orchestrator_1  | I1210 17:20:24.872637       1 rpc.go:147] Listening for RPC on :8935
broadcaster_1   | I1210 17:20:25.661966       1 accountmanager.go:99] Unlocked ETH account: 0x02452e64DA3959f7b3e487e27a8CD672808e49C2
broadcaster_1   | I1210 17:20:26.152509       1 livepeer.go:563] Maximum transcoding price per pixel is not greater than 0: 0, broadcaster is currently set to accept ANY price.
broadcaster_1   | I1210 17:20:26.152580       1 livepeer.go:564] To update the broadcaster's maximum acceptable transcoding price per pixel, use the CLI or restart the broadcaster with the appropriate 'maxPricePerUnit' and 'pixelsPerUnit' values
broadcaster_1   | I1210 17:20:26.195800       1 block_watcher.go:318] Backfilling block events (this can take a while)...
broadcaster_1   | I1210 17:20:26.195859       1 block_watcher.go:319] Start block: 5590611               End block: 5590610             Blocks elapsed: -1
broadcaster_1   | I1210 17:20:26.200135       1 block_watcher.go:537] fetching block logs from=5590611 to=5590610
transcoder_1    | I1210 17:20:26.245160       1 ot_rpc.go:50] Registering transcoder to orchestrator:8935
broadcaster_1   | I1210 17:20:26.245411       1 webserver.go:68] CLI server listening on broadcaster:7936
broadcaster_1   | I1210 17:20:26.246557       1 livepeer.go:742] ***Livepeer Running in Broadcaster Mode***
broadcaster_1   | I1210 17:20:26.247455       1 livepeer.go:743] Video Ingest Endpoint - rtmp://broadcaster:1935
orchestrator_1  | I1210 17:20:26.260520       1 ot_rpc.go:191] Got a RegisterTranscoder request from transcoder=172.21.0.3:44234 capacity=10
orchestrator_1  | I1210 17:20:26.870927       1 rpc.go:215] Connecting RPC to https://orchestrator:8935
orchestrator_1  | I1210 17:20:26.874302       1 rpc.go:187] Received Ping request
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
