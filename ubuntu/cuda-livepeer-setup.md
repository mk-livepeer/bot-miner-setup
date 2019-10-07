***GPU-ENABLED GO-LIVEPEER***

* While logged in as root, add a livepeer user:

```bash
adduser --disabled-password livepeer
```

Either fill in the user details, or press `<ENTER>` a few times to accept the defaults.

* Install `screen`:

```bash
apt-get install -y screen
```

* Impersonate the livepeer user:

```bash
su - livepeer
```

* Fetch go-livepeer using `go get`, then build livepeer and devtool:

```bash
git clone https://github.com/livepeer/go-livepeer "$HOME/go-livepeer"
cd "$HOME/go-livepeer"
go mod download
go build ./cmd/livepeer/livepeer.go
go build ./cmd/devtool/devtool.go
```

* Go-livepeer should be ready to go!  Exit the shell to return control back to the root user:

```bash
exit
```

--

Continue on to [ethminer-systemd-setup.md](ethminer-systemd-setup.md)
