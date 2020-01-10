# bot-miner-setup

Prometheus + Grafana setup

Included in this directory are example `docker-compose-*.yml` files.  To use them, copy the file that you want to use to a file named exactly, `docker-compose.yml`

Be sure to include `prometheus.yml` and all files within the `grafana` subdirectory in your deployment, or simply run it out of this directory.

Follow all instructions related to your setup in the corresponding documentation found in this repository, including setup of the eth passphrase and orch secret text files.

After bringing up the docker-compose system, access Grafana on port 3001 - http://localhost:3001
