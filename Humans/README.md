<p style="font-size:14px" align="right">
<a href="https://discord.gg/humansdotai" target="_blank">Join Humans discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Humans/humanslogo.jpg" width="30"/></a>
</p>


<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Humans/humanslogo.jpg">
</p>

# Humans node setup for Testnet — testnet-1

Official documentation:
>- [Validator setup instructions](https://docs.humans.zone/run-nodes/testnet/joining-testnet.html)

Explorer:
>-  [https://explorer.humans.zone/humans-testnet/)


## You can follow my [detailed guide](https://github.com/Vitek7373/testnet_manual/blob/main/Humans/Installation.md)

## Hardware Requirements
Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Recommended Hardware Requirements 
 - 4x CPUs; the faster clock speed the better
 - 8GB RAM
 - 200GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)


Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status
```
humansd status 2>&1 | jq .SyncInfo
```


### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
humansd keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase
```
humansd keys add $WALLET --recover
```

To get current list of wallets
```
humansd keys list
```

### Save wallet info
Add wallet and valoper address and load variables into the system
```
HUMANS_WALLET_ADDRESS=$(humansd keys show $WALLET -a)
HUMANS_VALOPER_ADDRESS=$(humansd keys show $WALLET --bech val -a)
echo 'export HUMANS_WALLET_ADDRESS='${HUMANS_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export HUMANS_VALOPER_ADDRESS='${HUMANS_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### Create validator
Before creating validator please make sure that you have at least 1 heart (1 heart is equal to 1000000 uheart) and your node is synchronized

To check your wallet balance:
```
humansd query bank balances $HUMANS_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
humansd tx staking create-validator \
  --amount 10000000uheart \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(humansd tendermint show-validator) \
  --moniker $NODENAME \
  --identity=<your_keybase_id> \
  --chain-id $HUMANS_CHAIN_ID \
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
sudo ufw allow ${HUMANS_PORT}656,${HUMANS_PORT}660/tcp
sudo ufw enable
```


### Get list of validators
```
humansd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids
```
curl -sS http://localhost:${HUMANS_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands
### Service management
Check logs
```
journalctl -fu humansd -o cat
```

Start service
```
sudo systemctl start humansd
```

Stop service
```
sudo systemctl stop humansd
```

Restart service
```
sudo systemctl restart humansd
```

### Node info
Synchronization info
```
humansd status 2>&1 | jq .SyncInfo
```

Validator info
```
humansd status 2>&1 | jq .ValidatorInfo
```

Node info
```
humansd status 2>&1 | jq .NodeInfo
```

Show node id
```
humansd tendermint show-node-id
```

### Wallet operations
List of wallets
```
humansd keys list
```

Recover wallet
```
humansd keys add $WALLET --recover
```

Delete wallet
```
humansd keys delete $WALLET
```

Get wallet balance
```
humansd query bank balances $HUMANS_WALLET_ADDRESS
```

Transfer funds
```
humansd tx bank send $HUMANS_WALLET_ADDRESS <TO_HUMANS_WALLET_ADDRESS> 10000000uheart --gas=auto
```

### Voting
```
humansd tx gov vote 1 yes --from $WALLET --chain-id=$HUMANS_CHAIN_ID --gas=auto
```

### Staking, Delegation and Rewards
Delegate stake
```
humansd tx staking delegate $HUMANS_VALOPER_ADDRESS 10000000uheart --from=$WALLET --chain-id=$HUMANS_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
humansd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uheart --from=$WALLET --chain-id=$HUMANS_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
humansd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$HUMANS_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
humansd tx distribution withdraw-rewards $HUMANS_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$HUMANS_CHAIN_ID --gas=auto
```

### Validator management
Edit validator
```
humansd tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$HUMANS_CHAIN_ID \
  --from=$WALLET \
  --gas=auto
```

Unjail validator
```
humansd tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$HUMANS_CHAIN_ID \
  --gas=auto
```

### Delete node
This commands will completely remove node from server.
```
sudo systemctl stop humansd
sudo systemctl disable humansd
sudo rm /etc/systemd/system/humans* -rf
sudo rm $(which humansd) -rf
sudo rm $HOME/.humansd* -rf
sudo rm $HOME/humans -rf
sed -i '/HUMANS_/d' ~/.bash_profile
```
