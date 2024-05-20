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
git clone https://github.com/UptickNetwork/uptick.git
cd uptick
git checkout v0.2.19
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
curl -o $HOME/.uptickd/config/genesis.json https://raw.githubusercontent.com/UptickNetwork/uptick-mainnet/master/uptick_117-1/genesis.json
```

## Set seeds and peers
```
SEEDS='e71bae28852a0b603f7360ec17fe91e7f065f324@uptick-mainnet-seed.itrocket.net:35656'
PEERS='dd482d080820020b144ca2efaf128d78261dea82@uptick-mainnet-peer.itrocket.net:10656,c7abddafe697b2a75a1567e0fe274d919e5fa404@65.109.106.214:15656,b12b37802e4000862ecd683a6f7eca6ef6daf569@65.109.60.19:26656,c6ff930a14586040cd9abfa58389d43aaca162a6@65.108.198.183:28656,fadab3eb04ebb651644ba15bb8f532bb509fe0f7@95.216.202.212:27656,2cc70e14c1cdb94edce3a9f8aa149880331af29d@212.23.222.109:26356,d3107602737ec267cd963672d14068b4f30fc633@213.239.207.175:26651,35ca65b1a865b0be132eb3212d9fd3a53be7d5da@176.9.183.45:26656,90c0c03d27e5b4354bffb709d28340f2657ca1c7@138.201.121.185:26679,7a320021212d346a7e8bfd5926feb4b307e7f69b@5.9.147.22:26556,f5224d6a57ec518cd427732de1e1d55732e91640@138.201.21.62:15656,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@142.132.134.181:30601,23e76540bea9b6851b92e280d7e0c123a0d49521@142.132.134.181:30599,bd2e1f218fde74045fbcff3fe36c467e7f05d7a3@198.244.165.50:21656,f058de92328b5a1ac44f45a4cd96850ebbca85bd@185.144.99.248:26656,d709d49fbf56dd9bf34463f15273a71d783c76cb@69.197.26.8:656,ada4a57a6eff26863d51847afe086544a6de1083@69.197.49.15:656,762152adcd6cc1f0537a3eded4043fc113078100@154.12.228.189:26656,7a762523ffc639de8d81d1ec40e180c6566e75db@142.132.156.99:3156,8ef5753cf3feba8f931ca771575d353556073e81@194.163.172.190:26656,e88413ee7153be8a9053165a60ad55492a8e300a@65.109.94.250:29656,ca698c533f814da69d2ad5ecd889b5790a189d05@184.174.36.212:36656,8fbfb8bff5d783df53b9ee95ab6b6e7ff708f280@65.108.134.215:32656,e8110e6c803fb4f16637ac76359fab7c605d4896@157.90.0.102:27656,3c40625cd7a8da2f27b178c1e69bcf2f1d4261a4@65.108.232.168:34656,8d9bfdb1e2657959ec641828080052d554fbe248@65.108.205.47:36656,ef9af846dcb2d25e7ccf5f7975a6d5d51fa01477@5.9.138.213:26656,446a4b3a6dcfc8f6c55dc02ce49e98936a713920@176.9.92.135:60756,91e671716f5af5e3bc7b491cf8c5933f725d4c9a@148.251.176.251:26656,34d86f3a8dfce7d8b615563c587433c65792f104@185.219.142.221:15656,c21eeb897d3fa45a81772b56038045d1d873252e@142.132.199.236:30656'
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
ExecStart=$(which uptickd) start --home $HOME/.uptickd --chain-id uptick_117-1
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
