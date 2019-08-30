***GPU-ENABLED GO-LIVEPEER***

* While logged in as root, add a livepeer user:

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

--

Continue on to [ethminer-systemd-setup.md](ethminer-systemd-setup.md)