<p style="font-size:14px" align="right">
<a href="https://discord.gg/initia" target="_blank">Join Initia discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Initia/initia.png" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Initia/initia.png">
</p>

# Uptick node setup for Testnet — initiation-1

Official documentation:
>- [Validator setup instructions](https://docs.initia.xyz/run-initia-node/running-initia-node/becoming-a-validator)

Explorer:
>-  https://scan.testnet.initia.xyz/initiation-1


## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Recommended Hardware Requirements 
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 200GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)


Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
initiad status 2>&1 | jq .SyncInfo
```


### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
initiad keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
initiad keys add $WALLET --recover
```

To get current list of wallets
```
initiad keys list
```

### Save wallet info
Add wallet and valoper address and load variables into the system
```
INITIA_WALLET_ADDRESS=$(initiad keys show $WALLET -a)
INITIA_VALOPER_ADDRESS=$(initiad keys show $WALLET --bech val -a)
echo 'export INITIA_WALLET_ADDRESS='${INITIA_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export INITIA_VALOPER_ADDRESS='${INITIA_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### Create validator
Before creating validator please make sure that you have at least 1 init (1 uptick is equal to 1000000 uinit) and your node is synchronized

To check your wallet balance:
```
initiad query bank balances $INITIA_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
initiad tx mstaking create-validator \
  --amount 1000000uinit \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --pubkey  $(initiad tendermint show-validator) \
  --moniker $NODENAME \
  --identity=<your_keybase_id> \
  --details="<your_validator_description>" \
  --chain-id $INITIA_CHAIN_ID \
  --gas=auto \
  --gas-prices 0.15uinit
```

## Security
To protect you keys please make sure you follow basic security rules

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
sudo ufw allow ${UPTICK_PORT}656,${UPTICK_PORT}660/tcp
sudo ufw enable
```


### Get list of validators
```
initiad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${INITIA_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu initiad -o cat
```

Start service
```
sudo systemctl start initiad
```

Stop service
```
sudo systemctl stop initiad
```

Restart service
```
sudo systemctl restart initiad
```

### Node info
Synchronization info
```
initiad status 2>&1 | jq .SyncInfo
```

Validator info
```
initiad status 2>&1 | jq .ValidatorInfo
```

Node info
```
initiad status 2>&1 | jq .NodeInfo
```

Show node id
```
initiad tendermint show-node-id
```

### Wallet operations
List of wallets
```
initiad keys list
```

Recover wallet
```
initiad keys add $WALLET --recover
```

Delete wallet
```
initiad keys delete $WALLET
```

Get wallet balance
```
initiad query bank balances $INITIA_WALLET_ADDRESS
```

Transfer funds
```
initiad tx bank send $INITIA_WALLET_ADDRESS <TO_INITIA_WALLET_ADDRESS> 5000uinit --gas=auto --gas-prices 0.15uinit
```

### Voting
```
initiad tx gov vote 1 yes --from $WALLET --chain-id=$INITIA_CHAIN_ID --gas=auto --gas-prices 0.15uinit
```

### Staking, Delegation and Rewards
Delegate stake
```
initiad tx mstaking delegate $INITIA_VALOPER_ADDRESS 5000000uinit --from=$WALLET --chain-id=$INITIA_CHAIN_ID --gas=auto --gas-prices 0.15uinit
```

Redelegate stake from validator to another validator
```
initiad tx mstaking redelegate <srcValidatorAddress> <destValidatorAddress> 5000000uinit --from=$WALLET --chain-id=$INITIA_CHAIN_ID --gas=auto --gas-prices 0.15uinit
```

Withdraw all rewards
```
initiad tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$INITIA_CHAIN_ID --gas=auto --gas-prices 0.15uinit
```

Withdraw rewards with commision
```
initiad tx distribution withdraw-rewards $INITIA_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$INITIA_CHAIN_ID --gas=auto --gas-prices 0.15uinit
```

### Validator management
Edit validator
```
initiad tx mstaking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$INITIA_CHAIN_ID \
  --from=$WALLET \
  --gas=auto
  --gas-prices 0.15uinit
```

Unjail validator
```
initiad tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$INITIA_CHAIN_ID \
  --gas=auto
  --gas-prices 0.15uinit
```

### Delete node
This commands will completely remove node from server.
```
sudo systemctl stop initiad
sudo systemctl disable initiad
sudo rm /etc/systemd/system/initia* -rf
sudo rm $(which initiad) -rf
sudo rm $HOME/.initiad* -rf
sudo rm $HOME/initia -rf
sed -i '/INITIA_/d' ~/.bash_profile
```
