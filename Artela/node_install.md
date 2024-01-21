<p style="font-size:14px" align="right">
<a href="https://discord.gg/artela" target="_blank">Join Artela discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Artela/logo%20artela.png" width="25"/></a>
</p>



<p align="center">
  <img height="130" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Artela/logo%20artela.png">
</p>

# Manual node setup "Mainnet"
If you want to setup fullnode manually follow the steps below

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<YOUR_MONIKER_VALIDATORS>
```

Save and import variables into system
```
ARTELA_PORT=22
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export ARTELA_CHAIN_ID=artela_11822-1" >> $HOME/.bash_profile
echo "export ARTELA_PORT=${ARTELA_PORT}" >> $HOME/.bash_profile
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
git clone https://github.com/artela-network/artela.git 
cd artela
git checkout v0.4.7-rc6
make install
```
## Checking the version, should show v0.4.7-rc6 
```
artelad version
```

## Config app
```
artelad config chain-id $ARTELA_CHAIN_ID
artelad config keyring-backend test
artelad config node tcp://localhost:${ARTELA_PORT}657
```

## Init app
```
artelad init $NODENAME --chain-id $ARTELA_CHAIN_ID
```

## Download genesis and addrbook
```
wget -qO $HOME/.artelad/config/genesis.json "https://github.com/Vitek7373/testnet_manual/blob/main/Artela/genesis.json"
```

## Set seeds and peers
```
SEEDS=""
PEERS="bec6934fcddbac139bdecce19f81510cb5e02949@47.254.24.106:26656,32d0e4aec8d8a8e33273337e1821f2fe2309539a@47.88.58.36:26656,1bf5b73f1771ea84f9974b9f0015186f1daa4266@47.251.14.47:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.artelad/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${ARTELA_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${ARTELA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${ARTELA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${ARTELA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${ARTELA_PORT}660\"%" $HOME/.artelad/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${ARTELA_PORT}317\"%; s%^address = \":8080\"%address = \":${ARTELA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${ARTELA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${ARTELA_PORT}091\"%" $HOME/.artelad/config/app.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="362880"
pruning_keep_every="0"
pruning_interval="100"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.artelad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.artelad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.artelad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.artelad/config/app.toml
```

## Set minimum gas price and timeout commit
```
export DENOM=uart
export CONFIG_DIR=$HOME/.artelad/config
sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.02'"${DENOM}"'"/' $CONFIG_DIR/app.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.artelad/config/config.toml
```

## Reset chain data
```
artelad tendermint unsafe-reset-all --home $HOME/.artelad
```

## Create service
```
sudo tee /etc/systemd/system/artelad.service > /dev/null <<EOF
[Unit]
Description=artela
After=network-online.target

[Service]
User=$USER
ExecStart=$(which artelad) start --home $HOME/.artelad
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
sudo systemctl enable artelad
sudo systemctl restart artelad && sudo journalctl -u artelad -f -o cat
```

