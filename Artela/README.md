<p style="font-size:14px" align="right">
<a href="https://discord.gg/artela" target="_blank">Join Artela discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Artela/logo%20artela.png" width="25"/></a>
</p>



<p align="center">
  <img height="120" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Artela/logo%20artela.png">
</p>

# Artela node setup for testnet — artela_11822-1

Official documentation:
>- [Validator setup instructions](https://docs.artela.network/develop/node/run-full-node)

Explorer:
>-  [Explorer](https://testnet.itrocket.net/artela)


## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Minimum Hardware Requirements
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 100GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

### Recommended Hardware Requirements 
 - 8x CPUs; the faster clock speed the better
 - 16GB RAM
 - 1TB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)



### Option (manual)
You can follow [manual guide](https://github.com/Vitek7373/testnet_manual/blob/main/Artela/node_install.md) if you better prefer setting up node manually

## Post installation

When installation is finished please load variables into system
```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
artelad status 2>&1 | jq .SyncInfo
```

### (OPTIONAL) State Sync
N/A

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
artelad keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
artelad keys add $WALLET --recover
```

To get current list of wallets
```
artelad keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
ARTELA_WALLET_ADDRESS=$(artelad keys show $WALLET -a)
ARTELA_VALOPER_ADDRESS=$(artelad keys show $WALLET --bech val -a)
echo 'export ARTELA_WALLET_ADDRESS='${ARTELA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export ARTELA_VALOPER_ADDRESS='${ARTELA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```



### Create validator
Before creating validator please make sure that you have at least 1 uart (1 uart is equal to 1000000 uart) and your node is synchronized

To check your wallet balance:
```
artelad query bank balances $ARTELA_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
artelad tx staking create-validator \
  --amount 100000000uart \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(artelad tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $ARTELA_CHAIN_ID
```


### Check your validator key
```
[[ $(artelad q staking validator $ARTELA_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(noisd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get list of validators
```
artelad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${ARTELA_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```


### Service management
Check logs
```
journalctl -fu artelad -o cat
```

Start service
```
sudo systemctl start artelad
```

Stop service
```
sudo systemctl stop artelad
```

Restart service
```
sudo systemctl restart artelad
```

### Node info
Synchronization info
```
artelad status 2>&1 | jq .SyncInfo
```

Validator info
```
artelad status 2>&1 | jq .ValidatorInfo
```

Node info
```
artelad status 2>&1 | jq .NodeInfo
```

Show node id
```
artelad tendermint show-node-id
```

### Wallet operations
List of wallets
```
artelad keys list
```

Recover wallet
```
artelad keys add $WALLET --recover
```

Delete wallet
```
artelad keys delete $WALLET
```

Get wallet balance
```
artelad query bank balances $ARTELA_WALLET_ADDRESS
```

Transfer funds
```
artelad tx bank send $ARTELA_WALLET_ADDRESS <TO_ARTELA_WALLET_ADDRESS> 10000000uart
```

### Voting
```
artelad tx gov vote 1 yes --from $WALLET --chain-id=$ARTELA_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
artelad tx staking delegate $ARTELA_VALOPER_ADDRESS 10000000uart --from=$WALLET --chain-id=$ARTELA_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
artelad tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uart --from=$WALLET --chain-id=$ARTELA_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
artelad tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$ARTELA_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
artelad tx distribution withdraw-rewards $ARTELA_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$ARTELA_CHAIN_ID
```

### Validator management
Edit validator
```
artelad tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$ARTELA_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
artelad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$ARTELA_CHAIN_ID \
  --gas=auto
```

