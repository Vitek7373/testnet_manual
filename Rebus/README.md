<p style="font-size:14px" align="right">
<a href="https://discord.gg/rebuschain" target="_blank">Join Rebus Discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/rebuslogo.png" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/rebuslogo.png">
</p>

# rebus node setup for mainnet — reb_1111-1

Official documentation:
>- https://github.com/rebuschain/rebus.mainnet/tree/master/reb_1111-1

Explorer:
>- https://rebus.explorers.guru


## Recommended Hardware Requirements 
 - 2x CPUs; the faster clock speed the better
 - 16GB RAM
 - 200GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)


Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
rebusd status 2>&1 | jq .SyncInfo
```

### Disable and cleanup indexing
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.rebusd/config/config.toml
sudo systemctl restart rebusd
sleep 3
sudo rm -rf $HOME/.rebusd/data/tx_index.db
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
rebusd keys add $WALLET
```

To recover your wallet using seed phrase
```
rebusd keys add $WALLET --recover
```

To get current list of wallets
```
rebusd keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
REBUS_WALLET_ADDRESS=$(rebusd keys show $WALLET -a)
REBUS_VALOPER_ADDRESS=$(rebusd keys show $WALLET --bech val -a)
echo 'export REBUS_WALLET_ADDRESS='${REBUS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export REBUS_VALOPER_ADDRESS='${REBUS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### Create validator
Before creating validator please make sure that you have at least 1 rebus (1 rebus is equal to 1000000000000000000 arebus) and your node is synchronized

To check your wallet balance:
```
rebusd query bank balances $REBUS_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
rebusd tx staking create-validator \
  --amount 1000000000000000000arebus \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.05" \
  --min-self-delegation "1" \
  --pubkey  $(rebusd tendermint show-validator) \
  --moniker $NODENAME \
  --identity=<your_keybase_id> \
  --chain-id $REBUS_CHAIN_ID
```

## Security
To protect you keys please make sure you follow basic security rules

### Set up ssh keys for authentication
Good tutorial on how to set up ssh keys for authentication to your server can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security
Start by checking the status of ufw.
```
sudo ufw status
```

Sets the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${REBUS_PORT}656,${REBUS_PORT}660/tcp
sudo ufw enable
```


### Check your validator key
```
[[ $(rebusd q staking validator $REBUS_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(rebusd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get list of validators
```
rebusd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${REBUS_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu rebusd -o cat
```

Start service
```
sudo systemctl start rebusd
```

Stop service
```
sudo systemctl stop rebusd
```

Restart service
```
sudo systemctl restart rebusd
```

### Node info
Synchronization info
```
rebusd status 2>&1 | jq .SyncInfo
```

Validator info
```
rebusd status 2>&1 | jq .ValidatorInfo
```

Node info
```
rebusd status 2>&1 | jq .NodeInfo
```

Show node id
```
rebusd tendermint show-node-id
```

### Wallet operations
List of wallets
```
rebusd keys list
```

Recover wallet
```
rebusd keys add $WALLET --recover
```

Delete wallet
```
rebusd keys delete $WALLET
```

Get wallet balance
```
rebusd query bank balances $REBUS_WALLET_ADDRESS
```

Transfer funds
```
rebusd tx bank send $REBUS_WALLET_ADDRESS <TO_REBUS_WALLET_ADDRESS> 1000000000000000000arebus
```

### Voting
```
rebusd tx gov vote 1 yes --from $WALLET --chain-id=$REBUS_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
rebusd tx staking delegate $REBUS_VALOPER_ADDRESS 1000000000000000000arebus --from=$WALLET --chain-id=$REBUS_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
rebusd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 1000000000000000000arebus --from=$WALLET --chain-id=$REBUS_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
rebusd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$REBUS_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
rebusd tx distribution withdraw-rewards $REBUS_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$REBUS_CHAIN_ID
```

### Validator management
Edit validator
```
rebusd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$REBUS_CHAIN_ID \
  --from=$WALLET
```

Unjail validator
```
rebusd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$REBUS_CHAIN_ID \
  --gas=auto
```
