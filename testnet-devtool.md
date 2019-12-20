***GO-LIVEPEER LOCAL TESTNET***

* Download and install docker, using the instructions [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04).

* Deploy a local testnet:

```bash
docker run -d -p 8545:8545 -p 8546:8546 --name geth-dev darkdragon/geth-with-livepeer-protocol:pm 
```

* Impersonate the `livepeer` user:

```bash
su - livepeer
```

* Export the library path:

```
export LD_LIBRARY_PATH=/usr/local/lib/
```

* Change into the `go-livepeer` directory:

```bash
cd "$HOME/go-livepeer"
```

* Start a screen session for the broadcaster:

```bash
screen
```

Hit `<ENTER>` a few times to get to a command prompt.

* Use the devtool to generate the appropriate command line for starting the broadcaster.  Run the following script, then copy and paste the last lines of its output to start the broadcaster:

```bash
go run cmd/devtool/devtool.go setup broadcaster 
```

For example:

```bash
./livepeer -v 99 -ethController {wallet} -datadir ./.lpdev2/broadcaster_{wallet} \
    -ethAcctAddr {wallet} \
    -ethUrl ws://localhost:8546/ \
    -ethPassword "" \
    -network=devenv \
    -monitor=false -currentManifest=true -cliAddr 127.0.0.1:7935 -httpAddr 127.0.0.1:8935  -broadcaster=true -rtmpAddr 127.0.0.1:1935 -depositMultiplier 200
```

NOTE: The devtool output may not include the `-depositMultiplier 200` argument.  You will have to add it yourself.

* Once the broadcaster is running, hold `<CTRL>` and type `<A>` then `<D>` to leave the screen session running in the background.

* Start another screen session for the transcoder:

```bash
screen
```

Hit `<ENTER>` a few times to get to a command prompt.

* Use the devtool to generate the appropriate command line for starting the transcoder.  Run the following script, then copy and paste the last lines of its output to start the transcoder:

```bash
go run cmd/devtool/devtool.go setup transcoder 
```

For example:

```bash
./livepeer -v 99 -ethController {wallet} -datadir ./.lpdev2/orchestrator_{wallet} \
    -ethAcctAddr {wallet} \
    -ethUrl ws://localhost:8546/ \
    -ethPassword "" \
    -network=devenv \
    -monitor=false -currentManifest=true -cliAddr 127.0.0.1:7936 -httpAddr 127.0.0.1:8936  -initializeRound=true \
    -serviceAddr 127.0.0.1:8936  -transcoder=true -orchestrator=true \
     -ipfsPath ./.lpdev2/orchestrator_{wallet}/trans -orchSecret secre -pricePerUnit 1 -nvidia 0
```

NOTE: The devtool output does not include the `-nvidia 0` argument.  You will have to add it yourself.  Use this argument to specify which GPU's to use.  To use the first two GPUs, specify, `-nvidia 0,1`.

* Once the transcoder is running, hold `<CTRL>` and type `<A>` then `<D>` to leave the screen session running in the background.

--

* At this point, you can log out if you like, or you can also log in with multiple windows if you wish to monitor both processes.

* You can list the screen sessions with the command:

```bash
screen -x
```

* To access the screen session, specify the session ID as an arg:

```bash
screen -x xxxx
```

You can always leave the screen session running in the background by holding `<CTRL>` and then typing `<A>` followed by `<D>`.

* stream into port 1935 using `ffmpeg`

```bash
ffmpeg -re -stream_loop -1 -i SomeVideo.mp4 -c:a copy -c:v copy -f flv rtmp://localhost:1935/movie
```

* playback from port 8935 using `ffplay`

```bash
ffplay http://localhost:8935/stream/movie.m3u8
```
