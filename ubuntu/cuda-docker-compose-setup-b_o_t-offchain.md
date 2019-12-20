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
    command: '-orchestrator -network offchain -orchSecret /secret.txt -serviceAddr orchestrator:8935 -orchAddr 0.0.0.0'
    ports:
      - 7935:7935
      - 8935:8935
    volumes:
      - osecret.txt:/secret.txt
  transcoder:
    depends_on:
      - orchestrator
    image: livepeer/go-livepeer:master
    command: '-transcoder -network offchain -orchAddr orchestrator:8935 -orchSecret /secret.txt -nvidia 0'
    volumes:
      - osecret.txt:/secret.txt
  broadcaster:
    depends_on:
      - orchestrator
      - transcoder
    image: livepeer/go-livepeer:master
    command: '-broadcaster -rtmpAddr broadcaster -orchAddr orchestrator:8935 -cliAddr broadcaster:7936 -httpAddr broadcaster:8936'
    ports:
      - 1935:1935
      - 7936:7936
      - 8936:8936
```

If you have multiple GPU's, adjust the `-nvidia` option on the transcoder container's command line to account for them.
For example, for two Nvidia GPUs, specify `-nvidia 0,1`
If you don't have any Nvidia GPUs, remove the `-nvidia 0` option from the transcoder container command line.

Hit `CTRL-X` to save and exit.  When prompted, hit the `Y` key followed by `ENTER`.

* Store your orchSecret password in a text file. Replace "secret" with a secret of your own:

```bash
echo secret > osecret.txt
```

* Start the B/O/T network using the following command:

```bash
sudo docker-compose up
```

You should see output similar to:

```
Starting botnetworkcomposed_orchestrator_1 ... 
Starting botnetworkcomposed_orchestrator_1 ... done
Starting botnetworkcomposed_transcoder_1 ... 
Starting botnetworkcomposed_transcoder_1 ... done
Starting botnetworkcomposed_broadcaster_1 ... 
Starting botnetworkcomposed_broadcaster_1 ... done
Attaching to botnetworkcomposed_orchestrator_1, botnetworkcomposed_transcoder_1, botnetworkcomposed_broadcaster_1
orchestrator_1  | I1006 18:42:24.729607       1 livepeer.go:205] ***Livepeer is running on the offchain*** network
orchestrator_1  | I1006 18:42:24.740214       1 livepeer.go:295] ***Livepeer is in off-chain mode***
orchestrator_1  | I1006 18:42:24.740609       1 webserver.go:65] CLI server listening on 127.0.0.1:7935
orchestrator_1  | I1006 18:42:24.741034       1 livepeer.go:682] ***Livepeer Running in Orchestrator Mode***
orchestrator_1  | I1006 18:42:24.741193       1 cert.go:83] Private key and cert not found. Generating
transcoder_1    | I1006 18:42:26.341787       1 livepeer.go:205] ***Livepeer is running on the offchain*** network
orchestrator_1  | I1006 18:42:24.761638       1 cert.go:22] Generating cert for orchestrator
orchestrator_1  | I1006 18:42:24.762918       1 rpc.go:152] Listening for RPC on :8935
broadcaster_1   | I1006 18:42:27.967617       1 livepeer.go:205] ***Livepeer is running on the offchain*** network
transcoder_1    | I1006 18:42:26.342908       1 livepeer.go:281] ***Livepeer is in transcoder mode ***
transcoder_1    | I1006 18:42:26.342943       1 ot_rpc.go:50] Registering transcoder to orchestrator:8935
orchestrator_1  | I1006 18:42:26.354784       1 ot_rpc.go:191] Got a RegisterTranscoder request from transcoder=172.18.0.3:45382 capacity=10
broadcaster_1   | I1006 18:42:27.969054       1 livepeer.go:295] ***Livepeer is in off-chain mode***
orchestrator_1  | I1006 18:42:26.741187       1 rpc.go:220] Connecting RPC to https://orchestrator:8935
orchestrator_1  | I1006 18:42:26.744904       1 rpc.go:192] Received Ping request
broadcaster_1   | I1006 18:42:27.969124       1 livepeer.go:684] ***Livepeer Running in Broadcaster Mode***
broadcaster_1   | I1006 18:42:27.969154       1 livepeer.go:685] Video Ingest Endpoint - rtmp://broadcaster:1935
broadcaster_1   | I1006 18:42:27.969268       1 webserver.go:65] CLI server listening on broadcaster:7936
```

Leave this running.

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
