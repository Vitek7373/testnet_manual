<p style="font-size:14px" align="right">
<a href="https://discord.gg/bGb6QSdgUS" target="_blank">Join Terp discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Terp-mainnet/terplogo.png" width="35"/></a>
</p>



<p align="center">
  <img height="130" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Terp-mainnet/terplogo.png">
</p>

# Manual node setup
If you want to setup fullnode manually follow the steps below

## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<Your alias, node name>
```

Save and import variables into system
```
TERP_PORT=31
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export TERP_CHAIN_ID=morocco-1" >> $HOME/.bash_profile
echo "export TERP_PORT=${TERP_PORT}" >> $HOME/.bash_profile
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
cd $HOME && rm -rf terp-core
git clone https://github.com/terpnetwork/terp-core.git
cd terp-core
git checkout v1.0.0
make install
```
## Checking the version
```
terpd version --long
```
 commit: 22f9b2992a9a113bff7b923f7f39c46ae0f61857
 cosmos_sdk_version: v0.47.1
 go: go version go1.20.2 linux/amd64
 name: terp
 server_name: terpd
 version: 1.0.0

## Config app
```
terpd config chain-id $TERP_CHAIN_ID
terpd config keyring-backend test
terpd config node tcp://localhost:${TERP_PORT}657
```

## Init app
```
terpd init $NODENAME --chain-id $TERP_CHAIN_ID
```

## Download genesis and addrbook
```
curl -s  https://raw.githubusercontent.com/terpnetwork/mainnet/main/morocco-1/genesis.json > ~/.terp/config/genesis.json
```
## Genesis sha256
```
sha256sum ~/.terp/config/genesis.json
```
The output should be like this: ab6c68c50d45cb9a145edf6b37c05cb9eefc2a0488d08498b8f827c2471ba843 genesis.json
## Set seeds and peers
```
SEEDS=""
PEERS="439f7a680cc645d888317cd64f9b8a6949de394b@65.109.154.185:26656,ccf0e37e2e5ae085b345c85e3adc4f57ffff739e@135.181.255.197:26656,f9b67e231c59b480e1f1f9ce158f166a4b9ee829@162.19.238.161:26656,da9a83ef835387e3813bd5cd79b1b0193f522d7c@65.21.152.68:26656,5c4d3ee03d080b3cb21a0b585e09da7ef56a82f3@192.81.208.147:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.terp/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${TERP_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${TERP_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${TERP_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${TERP_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${TERP_PORT}660\"%" $HOME/.terp/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${TERP_PORT}317\"%; s%^address = \":8080\"%address = \":${TERP_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${TERP_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${TERP_PORT}091\"%" $HOME/.terp/config/app.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.terp/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.terp/config/app.toml
```

## Set minimum gas price and timeout commit
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uthiol\"/" $HOME/.terp/config/app.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.terp/config/config.toml
```

## Reset chain data
```
terpd tendermint unsafe-reset-all --home $HOME/.terp
```

## Create service
```
sudo tee /etc/systemd/system/terpd.service > /dev/null <<EOF
[Unit]
Description=terp
After=network-online.target

[Service]
User=$USER
ExecStart=$(which terpd) start --home $HOME/.terp
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
sudo systemctl enable terpd
sudo systemctl restart terpd && sudo journalctl -u terpd -f -o cat
```
