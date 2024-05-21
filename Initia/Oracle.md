<p style="font-size:14px" align="right">
<a href="https://discord.gg/initia" target="_blank">Join Initia discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Initia/initia.png" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Initia/initia.png">
</p>

# Manual Oracle setup
If you want to setup Oracle manually follow the steps below


## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

## Download and build binaries
```
cd $HOME
git clone https://github.com/skip-mev/slinky.git
cd slinky
git checkout v0.4.3
make install
```

## Create service
```
sudo tee /etc/systemd/system/slinky.service > /dev/null <<EOF
[Unit]
Description=Initia Slinky Oracle
After=network-online.target

[Service]
User=$USER
ExecStart=$(which slinky) --oracle-config-path $HOME/slinky/config/core/oracle.json --market-map-endpoint 0.0.0.0:17090
Restart=on-failure
RestartSec=30
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable slinky
sudo systemctl restart slinky && sudo journalctl -u slinky -f -o cat
```

## Validating Prices
Upon launching the oracle, you should observe successful price retrieval from the provider sources. Additionally, you have the option to execute the test client script available in the Slinky repository by using the command:

```
make run-oracle-client
```
## Enable Oracle Vote Extension
In order to utilize the Slinky oracle data in the Initia node, the Oracle setting should be enabled in the config/app.toml file.

```
###############################################################################
###                                  Oracle                                 ###
###############################################################################
[oracle]
# Enabled indicates whether the oracle is enabled.
enabled = "true"

# Oracle Address is the URL of the out of process oracle sidecar. This is used to
# connect to the oracle sidecar when the application boots up. Note that the address
# can be modified at any point, but will only take effect after the application is
# restarted. This can be the address of an oracle container running on the same
# machine or a remote machine.
oracle_address = "127.0.0.1:21080"

# Client Timeout is the time that the client is willing to wait for responses from
# the oracle before timing out.
client_timeout = "500ms"

# MetricsEnabled determines whether oracle metrics are enabled. Specifically
# this enables instrumentation of the oracle client and the interaction between
# the oracle and the app.
metrics_enabled = "false"
```
