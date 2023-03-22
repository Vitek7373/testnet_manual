<p style="font-size:14px" align="right">
<a href="https://discord.gg/fFnQQnhw8z" target="_blank">Join Nois discord <img src="" width="30"/></a>
</p>



<p align="center">
  <img height="100" height="auto" src="">
</p>

# Nois node setup for testnet — nois-testnet-004

Official documentation:
>- [Validator setup instructions](https://docs2.nois.network/run_a_node.html)

Explorer:
>-  [Explorer](https://explorer.bccnodes.com/nois)


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
You can follow [manual guide](https://github.com/nois/manual_install.md) if you better prefer setting up node manually

## Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
noisd status 2>&1 | jq .SyncInfo
```

### (OPTIONAL) State Sync
N/A

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
noisd keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
noisd keys add $WALLET --recover
```

To get current list of wallets
```
noisd keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
NOIS_WALLET_ADDRESS=$(noisd keys show $WALLET -a)
NOIS_VALOPER_ADDRESS=$(noisd keys show $WALLET --bech val -a)
echo 'export NOIS_WALLET_ADDRESS='${NOIS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export NOIS_VALOPER_ADDRESS='${NOIS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```



### Create validator
Before creating validator please make sure that you have at least 1 nois (1 nois is equal to 1000000 unois) and your node is synchronized

To check your wallet balance:
```
noisd query bank balances $NOIS_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
noisd tx staking create-validator \
  --amount 100000000unois \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(noisd tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $NOIS_CHAIN_ID
```


### Check your validator key
```
[[ $(noisd q staking validator $NOIS_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(noisd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get list of validators
```
noisd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${NOIS_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```


### Service management
Check logs
```
journalctl -fu noisd -o cat
```

Start service
```
sudo systemctl start noisd
```

Stop service
```
sudo systemctl stop noisd
```

Restart service
```
sudo systemctl restart noisd
```

### Node info
Synchronization info
```
noisd status 2>&1 | jq .SyncInfo
```

Validator info
```
noisd status 2>&1 | jq .ValidatorInfo
```

Node info
```
noisd status 2>&1 | jq .NodeInfo
```

Show node id
```
noisd tendermint show-node-id
```

### Wallet operations
List of wallets
```
noisd keys list
```

Recover wallet
```
noisd keys add $WALLET --recover
```

Delete wallet
```
noisd keys delete $WALLET
```

Get wallet balance
```
noisd query bank balances $NOIS_WALLET_ADDRESS
```

Transfer funds
```
noisd tx bank send $NOIS_WALLET_ADDRESS <TO_NOIS_WALLET_ADDRESS> 10000000unois
```

### Voting
```
noisd tx gov vote 1 yes --from $WALLET --chain-id=$NOIS_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
noisd tx staking delegate $NOIS_VALOPER_ADDRESS 10000000unois --from=$WALLET --chain-id=$NOIS_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
noisd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000unois --from=$WALLET --chain-id=$NOIS_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
noisd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$NOIS_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
noisd tx distribution withdraw-rewards $NOIS_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$NOIS_CHAIN_ID
```

### Validator management
Edit validator
```
noisd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$NOIS_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
noisd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$NOIS_CHAIN_ID \
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
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" $HOME/.noisd/config/app.toml
sed -i -e "s/^snapshot-keep-recent *=.*/snapshot-keep-recent = \"$snapshot_keep_recent\"/" $HOME/.noisd/config/app.toml
```
