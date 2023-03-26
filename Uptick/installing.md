<p style="font-size:14px" align="right">
<a href="https://discord.gg/t3VxvyUapm" target="_blank">Join Uptick discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/upticklogo.png" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/upticklogo.png">
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
UPTICK_PORT=16
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export UPTICK_CHAIN_ID=uptick_117-1" >> $HOME/.bash_profile
echo "export UPTICK_PORT=${UPTICK_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux -y
```

## Install go
```
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

## Download and build binaries
```
cd $HOME
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.4
make install
```

## Config app
```
uptickd config chain-id $UPTICK_CHAIN_ID
uptickd config keyring-backend test
uptickd config node tcp://localhost:${UPTICK_PORT}657
```

## Init app
```
uptickd init $NODENAME --chain-id $UPTICK_CHAIN_ID
```

### Download Genesis
```
curl -o $HOME/.uptickd/config/genesis.json https://raw.githubusercontent.com/UptickNetwork/uptick-testnet/main/uptick_7000-2/genesis.json
```

## Set seeds and peers
```
SEEDS='f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@uptick.seed.brocha.in:30600'
PEERS='170397e75ca2b0f4e9f3b1bb5d0d23f9b10f01c7@uptick-sentry-1.p2p.brocha.in:30597,c0b33353fb70d8d71dcb9c8848b3b4207bd56951@uptick-sentry-2.p2p.brocha.in:30598,23e76540bea9b6851b92e280d7e0c123a0d49521@uptick-sentry-3.p2p.brocha.in:30599,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@uptick-rpc.p2p.brocha.in:30601,f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@uptick.seed.brocha.in:30600,48e7e8ca23b636f124e70092f4ba93f98606f604@54.37.129.164:55056,8ecd3260a19d2b112f6a84e0c091640744ec40c5@185.165.241.20:26656,8e924a598a06e29c9f84a0d68b6149f1524c1819@57.128.109.11:26656,05733da50967e3955e11665b1901d36291dfaee@65.108.195.30:21656,ed8af2e21ca079d722dd2222d93c18d18373401c@65.109.94.225:26656,d9bfa29e0cf9c4ce0cc9c26d98e5d97228f93b0b@uptick.rpc.kjnodes.com:15656,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@uptick.rpc.kjnodes.com:15659,90c0c03d27e5b4354bffb709d28340f2657ca1c7@138.201.121.185:26679,03d4bd74d72794fefc260008943d48dc502b7518@mainnet-uptick.konsortech.xyz:34656,d1e829e41b150445126e20dc91f4fc887d5be97d@65.109.112.29:28656,91c92,d4c8db525c22abf3ad005b228874cdab9e@54.174.83.27:26656,0b730506f313e19ae3b7d2815a33558ecefec066@uptick-peer-1.kalia.network:17656,14ca9d73314dd519bc0b0be8511c88f85fe6873e@uptick-peer-2.kalia.network:17656'
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.uptickd/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://0.0.0.0:26658\"%proxy_app = \"tcp://0.0.0.0:${UPTICK_PORT}658\"%; s%^laddr = \"tcp://0.0.0.0:26657\"%laddr = \"tcp://0.0.0.0:${UPTICK_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${UPTICK_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${UPTICK_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${UPTICK_PORT}660\"%" $HOME/.uptickd/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${UPTICK_PORT}317\"%; s%^address = \":8080\"%address = \":${UPTICK_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${UPTICK_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${UPTICK_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${UPTICK_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${UPTICK_PORT}546\"%" $HOME/.uptickd/config/app.toml
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.uptickd/config/config.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.uptickd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.uptickd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.uptickd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.uptickd/config/app.toml
```

## Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001auptick\"/" $HOME/.uptickd/config/app.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.uptickd/config/config.toml
```


## Create service
```
sudo tee /etc/systemd/system/uptickd.service > /dev/null <<EOF
[Unit]
Description=uptick
After=network-online.target

[Service]
User=$USER
ExecStart=$(which uptickd) start --home $HOME/.uptickd
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
sudo systemctl enable uptickd
sudo systemctl restart uptickd && sudo journalctl -u uptickd -f -o cat
```
