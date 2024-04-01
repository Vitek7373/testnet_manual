<p style="font-size:14px" align="right">
<a href="https://discord.gg/sideprotocol" target="_blank">Join Side Protocol discord <img src="logo.png" width="25"/></a>
</p>



<p align="center">
  <img height="120" height="auto" src="logo.png">
</p>

# Side node setup for testnet — side-testnet-3

Official documentation:
>- [Validator setup instructions](https://docs2.nois.network/run_a_node.html)

Explorer:
>-  [Explorer](https://testnet.side.explorers.guru/)


## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Minimum Hardware Requirements
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 100GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

### Recommended Hardware Requirements 
 - 8x CPUs; the faster clock speed the better
 - 64GB RAM
 - 1TB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)



### Option (manual)
You can follow [manual guide](https://github.com/Vitek7373/testnet_manual/blob/main/Side/node_install.md) if you better prefer setting up node manually

## Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
sided status 2>&1 | jq .SyncInfo
```

### (OPTIONAL) State Sync
N/A

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
sided keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
sided keys add $WALLET --recover
```

To get current list of wallets
```
sided keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
SIDE_WALLET_ADDRESS=$(sided keys show $WALLET -a)
SIDE_VALOPER_ADDRESS=$(sided keys show $WALLET --bech val -a)
echo 'export SIDE_WALLET_ADDRESS='${SIDE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export SIDE_VALOPER_ADDRESS='${SIDE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```



### Create validator
Before creating validator please make sure that you have at least 1 nois (1 side is equal to 1000000 uside) and your node is synchronized

To check your wallet balance:
```
sided query bank balances $SIDE_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
sided tx staking create-validator \
  --amount 100000000uside \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(sided tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $SIDE_CHAIN_ID
```


### Check your validator key
```
[[ $(sided q staking validator $SIDE_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(sided status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get list of validators
```
sided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${SIDE_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```


### Service management
Check logs
```
journalctl -fu sided -o cat
```

Start service
```
sudo systemctl start sided
```

Stop service
```
sudo systemctl stop sided
```

Restart service
```
sudo systemctl restart sided
```

### Node info
Synchronization info
```
sided status 2>&1 | jq .SyncInfo
```

Validator info
```
sided status 2>&1 | jq .ValidatorInfo
```

Node info
```
sided status 2>&1 | jq .NodeInfo
```

Show node id
```
sided tendermint show-node-id
```

### Wallet operations
List of wallets
```
sided keys list
```

Recover wallet
```
sided keys add $WALLET --recover
```

Delete wallet
```
sided keys delete $WALLET
```

Get wallet balance
```
sided query bank balances $SIDE_WALLET_ADDRESS
```

Transfer funds
```
sided tx bank send $SIDE_WALLET_ADDRESS <TO_SIDE_WALLET_ADDRESS> 10000000unois
```

### Voting
```
sided tx gov vote 1 yes --from $WALLET --chain-id=$SIDE_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
sided tx staking delegate $SIDE_VALOPER_ADDRESS 10000000uside --from=$WALLET --chain-id=$SIDE_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
sided tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uside --from=$WALLET --chain-id=$SIDE_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
sided tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$SIDE_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
sided tx distribution withdraw-rewards $SIDE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$SIDE_CHAIN_ID
```

### Validator management
Edit validator
```
sided tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$SIDE_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
sided tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$SIDE_CHAIN_ID \
  --gas=auto
```



### Pruning for state sync node
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="2000"
pruning_interval="50"
snapshot_interval="2000"
snapshot_keep_recent="5"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.side/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.side/config/app.toml
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" $HOME/.side/config/app.toml
sed -i -e "s/^snapshot-keep-recent *=.*/snapshot-keep-recent = \"$snapshot_keep_recent\"/" $HOME/.side/config/app.toml
```
