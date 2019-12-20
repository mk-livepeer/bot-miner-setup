***GO-LIVEPEER RINKEBY TESTNET***

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

* Run the following command to start the broadcaster:

```bash
./livepeer -v 99 ./livepeer -network rinkeby -broadcaster -orchAddr 127.0.0.1:8935 -cliAddr 127.0.0.1:7936 -httpAddr 127.0.0.1:8936 -depositMultiplier 1
```

* Once the broadcaster is running, hold `<CTRL>` and type `<A>` then `<D>` to leave the screen session running in the background.

* Start another screen session for the orchestrator:

```bash
screen
```

Hit `<ENTER>` a few times to get to a command prompt.

* Store your orchSecret password in a text file. Replace "secret" with a secret of your own:

```bash
echo secret > osecret.txt
```

* Run the following command to start the orchestrator:

```bash
./livepeer -v 99 -network rinkeby -orchestrator -orchSecret osecret.txt -pricePerUnit 1 -initializeRound=true -serviceAddr=127.0.0.1:8935 -orchAddr 0.0.0.0:8935
```

* Once the orchestrator is running, hold `<CTRL>` and type `<A>` then `<D>` to leave the screen session running in the background.

* Start another screen session for the transcoder:

```bash
screen
```

Hit `<ENTER>` a few times to get to a command prompt.

* Run the following command to start the transcoder:

```bash
./livepeer -v 99 -network rinkeby -transcoder -orchAddr 127.0.0.1:8935 -orchSecret osecret.txt -nvidia 0
```

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
