<p style="font-size:14px" align="right">
<a href="https://discord.gg/initia" target="_blank">Join Initia discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Initia/initia.png" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Initia/initia.png">
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
INITIA_PORT=17
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export INITIA_CHAIN_ID=initiation-1" >> $HOME/.bash_profile
echo "export INITIA_PORT=${INITIA_PORT}" >> $HOME/.bash_profile
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
  ver="1.22.2"
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
git clone https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.14
make install
```

## Config app
```
initiad config chain-id $INITIA_CHAIN_ID
initiad config keyring-backend test
initiad config node tcp://localhost:${INITIA_PORT}657
```

## Init app
```
initiad init $NODENAME --chain-id $INITIA_CHAIN_ID
```

### Download Genesis
```
curl -o $HOME/.initia/config/genesis.json https://initia.s3.ap-southeast-1.amazonaws.com/initiation-1/genesis.json
```

## Set seeds and peers
```
SEEDS='2eaa272622d1ba6796100ab39f58c75d458b9dbc@34.142.181.82:26656,c28827cb96c14c905b127b92065a3fb4cd77d7f6@testnet-seeds.whispernode.com:25756'
PEERS='aee7083ab11910ba3f1b8126d1b3728f13f54943@initia-testnet-peer.itrocket.net:11656,9f0ae0790fae9a2d327d8d6fe767b73eb8aa5c48@176.126.87.65:22656,8db26137b760df77c181b939100cdc5ec37c6879@84.46.242.223:15656,1d7d2d2cdb62df2a59aae536047d17f554e58bc3@154.38.181.13:656,1b0843bb3dce9c91115906305b698dc507bf138e@89.117.51.191:51656,72b8b9f0e826fa9be3f5ab55f56e67d409f0cef8@185.197.250.199:51656,ab948b87097b6474663e0132ac7360676f7030cd@62.169.26.15:26656,5a8e5f65179acb3d759099faccfe7752ca4ba536@178.18.248.75:26656,49da32b984143181ae5cae6564aba3a150624d7d@194.180.176.225:26656,b3b7823b530d47848e8f4d2f0cd2020b334bb001@161.97.139.248:16656,b027527aa552c6f292143a2adc61bfa23ecb3896@194.163.170.106:51656'
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.initia/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://0.0.0.0:26658\"%proxy_app = \"tcp://0.0.0.0:${INITIA_PORT}658\"%; s%^laddr = \"tcp://0.0.0.0:26657\"%laddr = \"tcp://0.0.0.0:${INITIA_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${INITIA_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${INITIA_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${INITIA_PORT}660\"%" $HOME/.initia/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${INITIA_PORT}317\"%; s%^address = \":8080\"%address = \":${INITIA_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${INITIA_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${INITIA_PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${INITIA_PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${INITIA_PORT}546\"%" $HOME/.initia/config/app.toml
```

## Disable indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.initia/config/config.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.initia/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.initia/config/app.toml
```

## Set minimum gas price
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.15uinit,0.01uusdc"|g' $HOME/.initia/config/app.toml
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.initia/config/config.toml
```


## Create service
```
sudo tee /etc/systemd/system/initiad.service > /dev/null <<EOF
[Unit]
Description=initia
After=network-online.target

[Service]
User=$USER
ExecStart=$(which initia) start --home $HOME/.initia
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
sudo systemctl enable initiad
sudo systemctl restart initiad && sudo journalctl -u initiad -f -o cat
```
