<p style="font-size:14px" align="right">
<a href="https://discord.gg/fFnQQnhw8z" target="_blank">Join Nois discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Nois/noislogo.png" width="25"/></a>
</p>



<p align="center">
  <img height="130" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Nois/noislogo.png">
</p>

# Manual node setup
If you want to setup fullnode manually follow the steps below

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<YOUR_MONIKER_VALIDATORS>
```

Save and import variables into system
```
NOIS_PORT=30
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export NOIS_CHAIN_ID=nois-testnet-004" >> $HOME/.bash_profile
echo "export NOIS_PORT=${NOIS_PORT}" >> $HOME/.bash_profile
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
  ver="1.20.2"
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
git clone https://github.com/noislabs/noisd.git 
cd noisd
git checkout v0.6.0
make install
```
## Checking the version, should show 0.6.0 
```
noisd version
```

## Config app
```
noisd config chain-id $NOIS_CHAIN_ID
noisd config keyring-backend test
noisd config node tcp://localhost:${NOIS_PORT}657
```

## Init app
```
noisd init $NODENAME --chain-id $NOIS_CHAIN_ID
```

## Download genesis and addrbook
```
wget -qO $HOME/.noisd/config/genesis.json "https://raw.githubusercontent.com/noislabs/networks/nois-testnet-004.final.2/nois-testnet-004/genesis.json"
```

## Set seeds and peers
```
SEEDS="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:17356,babc3f3f7804933265ec9c40ad94f4da8e9e0017@testnet-seed.rhinostake.com:17356"
PEERS="5ecd40831e453845587cbd03534e68a7b9fc3576@nois-testnet-peer.itrocket.net:21656,0154634ac7c81382fb2dc4017d1127a8ed0c0aec@65.108.254.238:26656,7bf392b33faa03a68c83c933623c84cdfbcb5a0e@178.63.8.245:60656,8aa378cf04c5eb13943c765c9b8f63b5f263b7e9@89.116.27.24:26956"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.noisd/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${NOIS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${NOIS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${NOIS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${NOIS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${NOIS_PORT}660\"%" $HOME/.noisd/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${NOIS_PORT}317\"%; s%^address = \":8080\"%address = \":${NOIS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${NOIS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${NOIS_PORT}091\"%" $HOME/.noisd/config/app.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.noisd/config/app.toml
```

## Set minimum gas price and timeout commit
```
export DENOM=unois
export CONFIG_DIR=$HOME/.noisd/config
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.00'"${DENOM}"'"/' $CONFIG_DIR/app.toml \
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
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.noisd/config/config.toml
```

## Reset chain data
```
noisd tendermint unsafe-reset-all --home $HOME/.noisd
```

## Create service
```
sudo tee /etc/systemd/system/noisd.service > /dev/null <<EOF
[Unit]
Description=nois
After=network-online.target

[Service]
User=$USER
ExecStart=$(which noisd) start --home $HOME/.noisd
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
sudo systemctl enable noisd
sudo systemctl restart noisd && sudo journalctl -u noisd -f -o cat
```
