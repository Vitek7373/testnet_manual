<p style="font-size:14px" align="right">
<a href="https://discord.gg/t3VxvyUapm" target="_blank">Join Uptick discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/upticklogo.png" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/upticklogo.png">
</p>

# Uptick node setup for Testnet — uptick_7000-2

Official documentation:
>- [Validator setup instructions](https://docs.uptick.network/testnet/)

Explorer:
>-  https://explorer.testnet.uptick.network/uptick-network-testnet


## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Recommended Hardware Requirements 
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 200GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)


Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
uptickd status 2>&1 | jq .SyncInfo
```


### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
uptickd keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
uptickd keys add $WALLET --recover
```

To get current list of wallets
```
uptickd keys list
```

### Save wallet info
Add wallet and valoper address and load variables into the system
```
UPTICK_WALLET_ADDRESS=$(uptickd keys show $WALLET -a)
UPTICK_VALOPER_ADDRESS=$(uptickd keys show $WALLET --bech val -a)
echo 'export UPTICK_WALLET_ADDRESS='${UPTICK_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export UPTICK_VALOPER_ADDRESS='${UPTICK_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### Create validator
Before creating validator please make sure that you have at least 1 uptick (1 uptick is equal to 1000000000000000000 auptick) and your node is synchronized

To check your wallet balance:
```
uptickd query bank balances $UPTICK_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
uptickd tx staking create-validator \
  --amount 5000000000000000000auptick \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(uptickd tendermint show-validator) \
  --moniker $NODENAME \
  --identity=<your_keybase_id> \
  --chain-id $UPTICK_CHAIN_ID \
  --gas=auto
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
uptickd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${UPTICK_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu uptickd -o cat
```

Start service
```
sudo systemctl start uptickd
```

Stop service
```
sudo systemctl stop uptickd
```

Restart service
```
sudo systemctl restart uptickd
```

### Node info
Synchronization info
```
uptickd status 2>&1 | jq .SyncInfo
```

Validator info
```
uptickd status 2>&1 | jq .ValidatorInfo
```

Node info
```
uptickd status 2>&1 | jq .NodeInfo
```

Show node id
```
uptickd tendermint show-node-id
```

### Wallet operations
List of wallets
```
uptickd keys list
```

Recover wallet
```
uptickd keys add $WALLET --recover
```

Delete wallet
```
uptickd keys delete $WALLET
```

Get wallet balance
```
uptickd query bank balances $UPTICK_WALLET_ADDRESS
```

Transfer funds
```
uptickd tx bank send $UPTICK_WALLET_ADDRESS <TO_UPTICK_WALLET_ADDRESS> 5000000000000000000auptick --gas=auto
```

### Voting
```
uptickd tx gov vote 1 yes --from $WALLET --chain-id=$UPTICK_CHAIN_ID --gas=auto
```

### Staking, Delegation and Rewards
Delegate stake
```
uptickd tx staking delegate $UPTICK_VALOPER_ADDRESS 5000000000000000000auptick --from=$WALLET --chain-id=$UPTICK_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
uptickd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 5000000000000000000auptick --from=$WALLET --chain-id=$UPTICK_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
uptickd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$UPTICK_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
uptickd tx distribution withdraw-rewards $UPTICK_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$UPTICK_CHAIN_ID --gas=auto
```

### Validator management
Edit validator
```
uptickd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$UPTICK_CHAIN_ID \
  --from=$WALLET \
  --gas=auto
```

Unjail validator
```
uptickd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$UPTICK_CHAIN_ID \
  --gas=auto
```

### Delete node
This commands will completely remove node from server.
```
sudo systemctl stop uptickd
sudo systemctl disable uptickd
sudo rm /etc/systemd/system/uptick* -rf
sudo rm $(which uptickd) -rf
sudo rm $HOME/.uptickd* -rf
sudo rm $HOME/uptick -rf
sed -i '/UPTICK_/d' ~/.bash_profile
```
