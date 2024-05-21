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
