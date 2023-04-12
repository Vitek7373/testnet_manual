<p style="font-size:14px" align="right">
<a href="https://discord.gg/bGb6QSdgUS" target="_blank">Join Terp discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/Terp-mainnet/terplogo.png" width="35"/></a>


<p align="center">
  <img height="150" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Terp-mainnet/terplogo.png">
</p>

# terp node setup for maineet — morocco-1

Official documentation:
>- [Validator setup instructions](https://github.com/terpnetwork/mainnet/tree/main/morocco-1)

Explorer:
>-  https://terp.zenscan.io/index.php


### Minimum Hardware Requirements
 - 4x CPUs; the faster clock speed the better
 - 16-64GB RAM
 - 500GB of storage (SSD or NVME)
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)


### Installing the node 
You can follow this [guide](https://github.com/Vitek7373/testnet_manual/blob/main/Terp-mainnet/installing.md)

## Post installation

When installation is finished please load variables into system

Next, you need to make sure that your node synchronizes the blocks. You can use command below to check synchronization status
```
terpd status 2>&1 | jq .SyncInfo
```

### Create wallet
To create new wallet you can use command below. Don’t forget to save the mnemonic
```
terpd keys add $WALLET
```

To recover your wallet using seed phrase
```
terpd keys add $WALLET --recover
```

To get current list of wallets
```
terpd keys list
```

### Save wallet info
Add wallet and valoper address into variables 
```
TERP_WALLET_ADDRESS=$(terpd keys show $WALLET -a)
TERP_VALOPER_ADDRESS=$(terpd keys show $WALLET --bech val -a)
echo 'export TERP_WALLET_ADDRESS='${TERP_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export TERP_VALOPER_ADDRESS='${TERP_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### Create validator
Before creating validator please make sure that you have at least 1 terp (1 terp is equal to 1000000 uterp) and your node is synchronized

To check your wallet balance:
```
terpd query bank balances $TERP_WALLET_ADDRESS
```
> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue 

To create your validator run command below
```
terpd tx staking create-validator \
  --amount 10000000uterp \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.06" \
  --min-self-delegation "1" \
  --pubkey  $(terpd tendermint show-validator) \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --identity=<your_keybase_id> \
  --moniker $NODENAME \
  --chain-id $TERP_CHAIN_ID
```

### Service management
Check logs
```
journalctl -fu terpd -o cat
```

Start service
```
sudo systemctl start terpd
```

Stop service
```
sudo systemctl stop terpd
```

Restart service
```
sudo systemctl restart terpd
```

### Node info
Synchronization info
```
terpd status 2>&1 | jq .SyncInfo
```

Validator info
```
terpd status 2>&1 | jq .ValidatorInfo
```

Node info
```
terpd status 2>&1 | jq .NodeInfo
```

Show node id
```
terpd tendermint show-node-id
```

### Wallet operations
List of wallets
```
terpd keys list
```

Recover wallet
```
terpd keys add $WALLET --recover
```

Delete wallet
```
terpd keys delete $WALLET
```

Get wallet balance
```
terpd query bank balances $TERP_WALLET_ADDRESS
```

Transfer funds
```
terpd tx bank send $TERP_WALLET_ADDRESS <TO_TERP_WALLET_ADDRESS> 10000000uterpx
```

### Voting
```
terpd tx gov vote 1 yes --from $WALLET --chain-id=$TERP_CHAIN_ID
```

### Staking, Delegation and Rewards
Delegate stake
```
terpd tx staking delegate $TERP_VALOPER_ADDRESS 10000000uterpx --from=$WALLET --chain-id=$TERP_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator
```
terpd tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000uterpx --from=$WALLET --chain-id=$TERP_CHAIN_ID --gas=auto
```

Withdraw all rewards
```
terpd tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$TERP_CHAIN_ID --gas=auto
```

Withdraw rewards with commision
```
terpd tx distribution withdraw-rewards $TERP_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$TERP_CHAIN_ID
```

