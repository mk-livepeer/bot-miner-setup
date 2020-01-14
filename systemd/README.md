***SYSTEMD***

We're going to run `go-livepeer` as a systemd service.  We will have to create a separate service file for each component of our B/O/T or B/OT network.

We can run the orchestrator and transcoder as separate services, or together as a single service.

Be sure to create the following files within the `/home/livepeer/go-livepeer/` directory:

* osecret.txt - contains your orchestrator secret

* pw.txt - contains your eth passphrase

To run as separate services:

* Create the `orchestrator.service` file:

```bash
vi /etc/systemd/system/orchestrator.service
```

* Push the `i` key to get into insertion mode.

* Paste the following:

```
[Unit]
Description=go-livepeer orchestrator daemon service
After=network.target

[Service]
User=livepeer
Group=livepeer
Type=simple
Restart=always
RestartSec=90s
Environment="LD_LIBRARY_PATH=/usr/local/lib/"
WorkingDirectory=/home/livepeer/go-livepeer/
ExecStart=/home/livepeer/go-livepeer/livepeer -network rinkeby -orchestrator -orchSecret osecret.txt -pricePerUnit 1 -initializeRound -serviceAddr 127.0.0.1:8935 -ethPassword pw.txt

[Install]
WantedBy=default.target
```

Update the `ExecStart` line to match your configuration. To use offline mode rather than the rinkeby testnet, replace the `ExecStart` line with the following:

```
ExecStart=/home/livepeer/go-livepeer/livepeer -network offchain -orchestrator -orchSecret osecret.txt -serviceAddr 127.0.0.1:8935
```

* Save the document and exit `vi` by first hitting the `ESC` key, then `:` followed by `wq` and `<ENTER>`

* Enable and start the services:

```bash
systemctl enable orchestrator.service
systemctl start orchestrator.service
```

* To monitor the service:

```bash
journalctl -f --unit=orchestrator.service
```

* Create the `transcoder.service` file:

```bash
vi /etc/systemd/system/transcoder.service
```

* Push the `i` key to get into insertion mode.

* Paste the following:

```
[Unit]
Description=go-livepeer transcoder daemon service
After=network.target

[Service]
User=livepeer
Group=livepeer
Type=simple
Restart=always
RestartSec=90s
Environment="LD_LIBRARY_PATH=/usr/local/lib/"
WorkingDirectory=/home/livepeer/go-livepeer/
ExecStart=/home/livepeer/go-livepeer/livepeer -network rinkeby -transcoder -orchAddr 127.0.0.1:8935 -orchSecret osecret.txt -nvidia 0

[Install]
WantedBy=default.target
```

Update the `ExecStart` line to match your configuration. To use offline mode rather than the rinkeby testnet, replace the `ExecStart` line with the following:

```
ExecStart=/home/livepeer/go-livepeer/livepeer -network offchain -transcoder -orchAddr 127.0.0.1:8935 -orchSecret osecret.txt -nvidia 0
```

* Save the document and exit `vi` by first hitting the `ESC` key, then `:` followed by `wq` and `<ENTER>`

* Enable and start the services:

```bash
systemctl enable transcoder.service
systemctl start transcoder.service
```

* To monitor the service:

```bash
journalctl -f --unit=transcoder.service
```

To run the orchestrator and transcoder as a single combined service:

* Create the `orchtran.service` file:

```bash
vi /etc/systemd/system/orchtran.service
```

* Push the `i` key to get into insertion mode.

* Paste the following:

```
[Unit]
Description=go-livepeer orchestrator+transcoder daemon service
After=network.target

[Service]
User=livepeer
Group=livepeer
Type=simple
Restart=always
RestartSec=90s
Environment="LD_LIBRARY_PATH=/usr/local/lib/"
WorkingDirectory=/home/livepeer/go-livepeer/
ExecStart=/home/livepeer/go-livepeer/livepeer -network rinkeby -orchestrator -transcoder -pricePerUnit 1 -nvidia 0 -initializeRound -serviceAddr 127.0.0.1:8935 -ethPassword pw.txt

[Install]
WantedBy=default.target
```

Update the `ExecStart` line to match your configuration. To use offline mode rather than the rinkeby testnet, replace the `ExecStart` line with the following:

```
ExecStart=/home/livepeer/go-livepeer/livepeer -network offchain -orchestrator -transcoder -serviceAddr 127.0.0.1:8935 -nvidia 0
```

* Save the document and exit `vi` by first hitting the `ESC` key, then `:` followed by `wq` and `<ENTER>`

* Enable and start the services:

```bash
systemctl enable orchtran.service
systemctl start orchtran.service
```

* To monitor the service:

```bash
journalctl -f --unit=orchtran.service
```

To run a broadcaster node service:

* Create the `broadcaster.service` file:

```bash
vi /etc/systemd/system/broadcaster.service
```

* Push the `i` key to get into insertion mode.

* Paste the following:

```
[Unit]
Description=go-livepeer broadcaster daemon service
After=network.target

[Service]
User=livepeer
Group=livepeer
Type=simple
Restart=always
RestartSec=90s
Environment="LD_LIBRARY_PATH=/usr/local/lib/"
WorkingDirectory=/home/livepeer/go-livepeer/
ExecStart=/home/livepeer/go-livepeer/livepeer -network rinkeby -broadcaster -orchAddr 127.0.0.1:8935 -cliAddr :7936 -httpAddr 127.0.0.1:8936 -ethPassword pw.txt

[Install]
WantedBy=default.target
```

Update the `ExecStart` line to match your configuration. To use offline mode rather than the rinkeby testnet, replace the `ExecStart` line with the following:

```
ExecStart=/home/livepeer/go-livepeer/livepeer -network offchain -broadcaster -orchAddr 127.0.0.1:8935 -cliAddr :7936  -httpAddr 127.0.0.1:8936
```

* Save the document and exit `vi` by first hitting the `ESC` key, then `:` followed by `wq` and `<ENTER>`

* Enable and start the services:

```bash
systemctl enable broadcaster.service
systemctl start broadcaster.service
```

* To monitor the service:

```bash
journalctl -f --unit=broadcaster.service
```
