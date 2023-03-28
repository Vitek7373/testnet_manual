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
echo "export NOIS_CHAIN_ID=nois-1" >> $HOME/.bash_profile
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
git checkout v1.0.0
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
wget -qO $HOME/.noisd/config/genesis.json "https://raw.githubusercontent.com/noislabs/networks/nois1.final.1/nois-1/genesis.json"
```

## Set seeds and peers
```
SEEDS=""
PEERS="PEERS=3c5926d0b4b8750f16f6495063e6d762b2556d1e@65.21.122.47:27656,f9c01cefd0f119b29b72c96bd84f37bb9d273874@65.108.6.45:61456,2e1d9305a5be27fc708ea7bc2fade939be1259e6@65.108.82.62:51656,e84cbe410271d84b2968c46881522bd3e9726898@144.76.30.36:15663,08f825cbea77a7e4e23f05d7372e3e6c97dfcca5@65.108.224.156:26656,5c75e139557a96bc660f39c58568b07943922c48@159.148.146.132:26656,91952caf57ac1c26738f069d2e79ca9b539f641d@135.181.198.17:26656,7502abfa0929a2469f10696f6f309c7e7c5555ab@95.217.83.28:17356,5dbe16d3eb7a0d0ef74aacb7c1958af6072d3644@65.108.66.34:27756,871066c94ef32f061f5f3db4d7a6d94b38d73c0f@65.109.92.240:40136,23d7872bdd8b1bf80b52cb20da57b88a4935bc3d@65.109.30.197:22656,1d3861fb38164385d5b98c4cf4e35452bab403cc@149.102.146.216:26656,7e7c9d5a5b575f00f82a960db608284854cf4c4d@85.10.207.188:26656,374615fcb23cfbd30a59a2b904cf675d9b93b7e0@78.46.61.117:01656,ae02b0a36568a1f2be71bd98840aae333d1e3147@51.159.195.168:46656,78c9915ae359907266e0eb713b911bdae21b4876@136.243.103.32:26656,c10bacf94b9a70fa57acfa1aaa4498b84eb4109e@195.201.243.40:17356,b5058b5422c6bdba55eafac46cc23731288f42c8@130.255.170.126:26656,c695f41458b08fe87729beffa513f1c38d20d1db@193.70.33.64:17356,ed0cce5194ebefdf2f4d9301efc9a12101c35aa2@57.128.163.232:26656,9d21af60ad2568ffcb55a0bd0eb03b6cfa2644c5@49.12.120.113:26656,9741a97c632f623cdbbd8a91ef4bd18bfd01059a@5.9.79.121:17656,8cce0e919b1a7c42086a712748c8e84d7d7cd9ac@135.181.155.14:26656,d4db7bb58fda20fdba8b3b752cd5d15d68ec7980@54.91.95.247:26656,0cf59ab91e4a96d6e5427d903644edd18d9421d1@142.132.248.138:26786,763f4cd38f0685616b6657d9a34c1cdbf01ca90c@212.23.222.109:26456,fefa1d14781af7cd0067c3fe14f8c119cc9afadf@38.146.3.180:17356,178c52ade4c56f40110766fee09642513ee12a9c@216.128.134.36:17356,d4f30672ef58f234fd13b503f7ca3d32ffc4e7a2@45.63.104.164:26656,eeb51b9e6c7d6de977e3c6419f3bba78263b4b7e@192.99.32.49:26656,1f11007b46c24a24cdceba685e6c47d783ba2a09@46.4.50.247:51656,891d736102e005c83147af14f7a4819dcb5bccd2@65.21.62.231:19656,b26e5ac4afbadf96ad31ee3aeb5e6557f2894037@65.108.199.222:30656,0371e0701ba6aaf231d147d49cdd67735d64495a@65.109.104.118:60656,ca92abdc4599dd91dd63e689c64c468df5425f2c@95.216.100.99:51656,dac094230205ee1f49b42ac0ace9d95c3578d7e7@88.198.18.88:32656,08db9ae3198ac1e0fe33333dcaec949a274b6d75@95.31.16.222:26656,497dff4750970f8d142c9c61da4acee0e3ff76c4@141.95.155.224:12156"
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
