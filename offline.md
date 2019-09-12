***GO-LIVEPEER TESTNET***


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

* Start an offline orchestrator on port 8935 with a secret.  Replace "test" with a secret of your own:

```bash
./livepeer -orchestrator --network offchain -orchSecret test -serviceAddr 127.0.0.1:8935 -orchAddr 0.0.0.0:8935
```

* Once the broadcaster is running, hold `<CTRL>` and type `<A>` then `<D>` to leave the screen session running in the background.

* Start another screen session for the transcoder:

```bash
screen
```

Hit `<ENTER>` a few times to get to a command prompt.

* Start an offline transcoder connecting to the orchestrator with secret "test".  Replace "test" with your own secret:

```bash
./livepeer -transcoder -network offchain -orchAddr 127.0.0.1:8935 -orchSecret test -nvidia 0
```

NOTE: Use the `-nvidia` argument to control the GPUs being used.  To use the first two GPUs, specify, `-nvidia 0,1`

* Once the transcoder is running, hold `<CTRL>` and type `<A>` then `<D>` to leave the screen session running in the background.

* Start another screen session for the broadcaster:

```bash
screen
```

Hit `<ENTER>` a few times to get to a command prompt.

* Start an offline broadcaster on port 8936, connected to the orchestrator on port 8935. A secret is not necessary here because the secret is a Orchestrator / Transcoder concept):

```bash
./livepeer -broadcaster -orchAddr 127.0.0.1:8935 -cliAddr 127.0.0.1:7936  -httpAddr 127.0.0.1:8936
```

* Once the broadcaster is running, hold `<CTRL>` and type `<A>` then `<D>` to leave the screen session running in the background.

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

* playback from port 8936 using `ffplay`

```bash
ffplay http://localhost:8936/stream/movie.m3u8
```