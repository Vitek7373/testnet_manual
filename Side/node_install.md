<p style="font-size:14px" align="right">
<a href="https://discord.gg/sideprotocol" target="_blank">Join Side Protocol discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Side/logo.png" width="50"/></a>
</p>



<p align="center">
  <img height="130" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Side/logo.png">
</p>

# Manual node setup "Testnet"
If you want to setup fullnode manually follow the steps below

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<YOUR_MONIKER_VALIDATORS>
```

Save and import variables into system
```
SIDE_PORT=27
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export SIDE_CHAIN_ID=S2-testnet-2" >> $HOME/.bash_profile
echo "export SIDE_PORT=${SIDE_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```

## Install go
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.21.3"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
git clone -b dev https://github.com/sideprotocol/sidechain
cd sidechain
git checkout v0.8.1
make install
```
## Checking the version, should show 0.8.1
```
sided version
```

## Config app
```
sided config chain-id $SIDE_CHAIN_ID
sided config keyring-backend os
sided config node tcp://localhost:${SIDE_PORT}657
```

## Init app
```
sided init $NODENAME --chain-id $SIDE_CHAIN_ID
```

## Download genesis and addrbook
```
wget -qO $HOME/.side/config/genesis.json "https://github.com/sideprotocol/testnet/raw/main/S2-testnet-2/genesis.json"
```

## Set seeds and peers
```
SEEDS=""
PEERS="43cb99189637d1e35b8f11c1580cff305157c94b@54.249.68.205:26656,519453a49e25826c04c9d2779ec7b5971876665d@138.201.51.154:32004,6aa033b16b4eea79195febbf87fd21f51b1a1bde@46.4.55.46:20656,df3cb7c20c0ca87926a06424a4aebe5cc485301d@54.39.107.180:45656,70c139e670cbd4456cdca20953960e600e90c9f2@65.108.68.214:45656,db6fc589d5db96b5ff4e733c16afd4a00488ae96@168.119.10.134:22956,af499b4f78ac7ecf0c340242b973d7592e98db62@213.199.39.54:45656,a5fd292715327ca65a8c305fe176166cbe0b8207@146.59.53.93:45656,59fa36770ef7f6cdbb4fe9c70c13b501c1a6b258@95.214.55.138:4656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.side/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${SIDE_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${SIDE_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${SIDE_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${SIDE_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${SIDE_PORT}660\"%" $HOME/.side/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${SIDE_PORT}317\"%; s%^address = \":8080\"%address = \":${SIDE_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${SIDE_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${SIDE_PORT}091\"%" $HOME/.side/config/app.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.side/config/app.toml
```

## Set minimum gas price and timeout commit
```
export DENOM=uside
export CONFIG_DIR=$HOME/.side/config
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.005'"${DENOM}"'"/' $CONFIG_DIR/app.toml \
  && sed -i 's/^timeout_propose =.*$/timeout_propose = "2s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_propose_delta =.*$/timeout_propose_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_prevote =.*$/timeout_prevote = "1s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_prevote_delta =.*$/timeout_prevote_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_precommit =.*$/timeout_precommit = "1s"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_precommit_delta =.*$/timeout_precommit_delta = "500ms"/' $CONFIG_DIR/config.toml \
  && sed -i 's/^timeout_commit =.*$/timeout_commit = "2s"/' $CONFIG_DIR/config.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.side/config/config.toml
```

## Reset chain data
```
sided tendermint unsafe-reset-all --home $HOME/.side
```

## Create service
```
sudo tee /etc/systemd/system/sided.service > /dev/null <<EOF
[Unit]
Description=side
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sided) start --home $HOME/.side
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service
```
sudo systemctl daemon-reload
sudo systemctl enable sided
sudo systemctl restart sided && sudo journalctl -u sided -f -o cat
```
