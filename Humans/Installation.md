<p style="font-size:14px" align="right">
<a href="https://discord.gg/humansdotai" target="_blank">Join Humans discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Humans/humanslogo.jpg" width="30"/></a>
</p>



<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Humans/humanslogo.jpg">
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
HUMANS_PORT=22
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export HUMANS_CHAIN_ID=testnet-1" >> $HOME/.bash_profile
echo "export HUMANS_PORT=${HUMANS_PORT}" >> $HOME/.bash_profile
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
git clone https://github.com/humansdotai/humans
cd humans
git checkout v1.0.0
go build -o humansd cmd/humansd/main.go
sudo cp humansd $HOME/go/bin/humansd
```

## Config app
```
humansd config chain-id $HUMANS_CHAIN_ID
humansd config keyring-backend test
humansd config node tcp://localhost:${HUMANS_PORT}657
```

## Init app
```
humansd init $NODENAME --chain-id $HUMANS_CHAIN_ID
```

## Download genesis and addrbook
```
cd $HOME && curl -s https://rpc-testnet.humans.zone/genesis | jq -r .result.genesis > genesis.json
cp genesis.json $HOME/.humans/config/genesis.json
```

## Set seeds and peers
```
SEEDS=""
PEERS="09af742cbdb91cad85dad8d6cf17d2dc178fe4ba@167.235.97.139:26656,aac683209559ca9ea48de4c47f3806483a5ec13f@185.244.180.97:26656,bc0677f947c43f676b86fce1cfd063aa48580aad@172.104.76.251:26656,5e41a64298ca653af5297833c6a47eb1ad1bf367@154.38.161.212:36656,5e9a778625c2d4f2ffa0691f50342a58e8b9bfdd@45.136.40.14:26656,1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,1a06290cf2da4a2cf513036180d250035266b109@45.136.40.20:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,6e7fb6dff31da195d4fd92cdfbf049fc0ddc8403@45.136.40.21:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656,0794ff9952e1b6c6ead96359fecc52932e115eab@45.136.40.15:26656,d55876bc04e363bbe68a7fb344dd65632e310f45@138.201.121.185:26668,9e83df8d212ae04fd67d24b9f45971887d28d423@65.109.85.170:45656,9a20e4a0c977950db219770c1523a1b6182af367@79.143.179.196:46656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,1802da0f18dcf6752ed789d81b4f5e003f80719b@45.136.40.19:26656,2cc7701b7d2a9e0384ad787061edd4e5e63357d3@65.109.34.41:26656,3f13ad6e8795479b051d147a5049bf4bd0a63817@65.108.142.47:22656,e9ba9556b7076a679ea117b44ef18eeb59dc8f61@45.136.40.17:26656,547e5c5573f9f3077abf87d942d5fe74be0548df@65.109.38.55:26656,2cc5eb1ce7fe9744a9938611e0f3a47d3cc8b195@65.109.54.15:15656,c39257353508f74c5028efa5b4290561580ac4c1@164.68.102.193:26656,fb41827ecf20d787b02a40c9b002c01c14a80245@81.196.190.108:20656,25adac50f326015fe5434d94ef9ffbcbd2bd062e@51.158.66.152:26656,cff69c40f95479a5655b255c8cc97a096ec91e6f@65.109.58.29:26656,2964169c08736f09ac3a7767122edb22ac4ad9c6@176.57.189.212:46656,69822c67487d4907f162fdd6d42549e1df60c82d@65.21.224.248:26656,f56cb3a2c6dbfa52a5b17ef18aab2c8beadc89c3@84.46.246.102:2556,e298f79eaac5bcdb157f1eeca8dc05f2293402b7@75.119.134.86:26656,17f4b40a52cb18293edc4f3c13e33efd09f446d4@65.109.53.60:26656,e454468a9134e6c44e06022a35b1b504226cceb5@194.163.169.166:2556,305d97dd4f8e459980b499e24e6a55213cec2273@185.249.227.102:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.humans/config/config.toml
```

## Set custom ports
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${HUMANS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${HUMANS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${HUMANS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${HUMANS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${HUMANS_PORT}660\"%" $HOME/.humans/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${HUMANS_PORT}317\"%; s%^address = \":8080\"%address = \":${HUMANS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${HUMANS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${HUMANS_PORT}091\"%" $HOME/.humans/config/app.toml
```

## Config pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.humans/config/app.toml
```

## Set minimum gas price and timeout commit
```
CONFIG_TOML="$HOME/.humans/config/config.toml"
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025uheart"/g' $HOME/.humans/config/app.toml \
  && sed -i 's/timeout_propose =.*/timeout_propose = "100ms"/g' $CONFIG_TOML \
  && sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML \
  && sed -i 's/timeout_prevote =.*/timeout_prevote = "100ms"/g' $CONFIG_TOML \
  && sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML \
  && sed -i 's/timeout_precommit =.*/timeout_precommit = "100ms"/g' $CONFIG_TOML \
  && sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML \
  && sed -i 's/timeout_commit =.*/timeout_commit = "1s"/g' $CONFIG_TOML \
  && sed -i 's/skip_timeout_commit =.*/skip_timeout_commit = false/g' $CONFIG_TOML
```

## Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.humans/config/config.toml
  ```

## Reset chain data
```
humansd tendermint unsafe-reset-all --home $HOME/.humans
```

## Create service
```
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humans
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start --home $HOME/.humans
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
sudo systemctl enable humansd
sudo systemctl restart humansd && sudo journalctl -u humansd -f -o cat
```
