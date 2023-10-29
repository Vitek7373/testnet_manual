<p style="font-size:14px" align="right">
<a href="https://discord.gg/d27tSnnNSu" target="_blank">Join Avail discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Avail/availlogo.png" width="25"/></a>
</p>



<p align="center">
  <img height="120" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Avail/availlogo.png">
</p>

# Avail node setup for testnet — Kate Testnet

Official documentation:
>- [Validator setup instructions](https://docs.availproject.org/build/quickstart/)

Explorer:
>-  [Explorer](https://telemetry.avail.tools/)



### Minimum Hardware Requirements
 - 2 core CPUs; the faster clock speed the better
 - 4GB RAM
 - 20GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

### Recommended Hardware Requirements 
 - 4 core CPUs; the faster clock speed the better
 - 8GB RAM
 - 300GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)


## Setting up vars
Here you have to put name of your moniker (validator) that will be visible in explorer
```
NODENAME=<YOUR_MONIKER_VALIDATORS>
```
Save and import variables into system
```
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
echo "export AVAIL_CHAIN_ID=kate" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl tar wget clang pkg-config protobuf-compiler libssl-dev jq build-essential protobuf-compiler bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
## Install Rust
```
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```

## Download and build binaries
```
git clone https://github.com/availproject/avail.git
cd avail
cargo build --release -p data-avail
mkdir -p output
git checkout v1.7.2
cargo run --locked --release -- --chain kate -d ./output
```
## Create service
```
sudo tee /etc/systemd/system/availd.service > /dev/null <<EOF
[Unit]
Description=Avail
After=network-online.target

[Service]
User=$USER
ExecStart=/root/avail/target/release/data-avail --base-path `pwd`/data --chain $AVAIL_CHAIN_ID --name "$NODENAME"
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
