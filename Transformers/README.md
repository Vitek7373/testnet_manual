<p style="font-size:14px" align="right">
<a href="https://discord.gg/6PVYyQnRCy" target="_blank">Join Transformers discord <img src="https://github.com/Vitek7373/testnet_manual/blob/main/transformerslogo.png" width="30"/></a>
</p>

<p align="center">
  <img height="200" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/transformerslogo.png">
</p>

Transformers Chain is a high-performance distributed system with continuously scalable transaction capability, based on a special Raindrop consensus protocol (RDCP), which realises the ability of multiple block producers to process chain transactions in parallel, and the concurrent number of its block producers can be continuously increased with the scale of the network, through its incentive layer protocol network realises a decentralised free development state.

#  Transformers node setup for Testnet

Official website:
>-  https://www.tfsc.io/#/pc/Index

Explorer:
>-  https://explorer.tfsc.io/#/pc/index



### Minimum Hardware Requirements
 - 8x CPUs; the faster clock speed the better
 - 16GB RAM
 - 100GB Disk
 - Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

## Update packages
```
sudo apt update && sudo apt upgrade -y
```

## Install dependencies
```
sudo apt install curl build-essential git wget jq make gcc tmux -y
```
## Software requirements：
Operation system：Run on CentOS 7、ubuntu-22.04-desktop-amd64 system.

## Command-line mode
Download the main network program（ttfsc_0.x.x_dev.zip）, then unpack to get the ttfsc_0.x.x_dev binary executable, and then modify the execution permission of the program.

## Creating a new directory
```
cd $HOME && mkdir .transformers
```

## Download binary executable
```
cd $HOME/.transformers && wget -q https://fastcdn.uscloudmedia.com/transformers/test/tfs_v0.22.0_bb350da_devnet.tar && tar -xvf tfs_v0.22.0_bb350da_devnet.tar
```
## We set the rights

```
chmod +x $HOME/.transformers/tfs_v0.22.0_bb350da_devnet.tar
```

## Run the node with the -c flag so that it initializes and creates a config.json file:
```
cd $HOME/.transformers && ./tfs_v0.22.0_bb350da_devnet.tar -c
```

## Now you can run the node.

Create a new tmux session named tfsc:
```
tmux new-session -s tfsc
```

In the opened session, we will launch the node with the -m flag:
```
cd $HOME/.transformers && ./tfs_v0.22.0_bb350da_devnet.tar -m
```

### Something like this will appear, and logs will go a little later:

<p align="center">
  <img height="100" height="auto" src="https://github.com/Vitek7373/testnet_manual/blob/main/Transformers/scrine1.png">
</p>

Minimize the session with ctrl+ b, and then d

## To reconnect to the session
```
tmux attach
```

### Interaction with the node
To find out information about the node, press 7 and enter. You can see the node version, wallet address (Base58), balance, synchronization (block top) and gas cost.

1. Transaction
Press 1 to enter the transaction interface.

# Enter the account number address of the transaction initiator.

input FromAddr: 12GwpCQi7bWr8cbmU2r1aFia1rUQJDVXdo

# Enter the account address of the other party:

input ToAddr: 1vkS46QffeM4sDMBBjuJBiVkMQKY7Z8Tu


# Enter the transaction amount
input amount: 10


## Next, make a backup of the key file
The wallet file is called 153VtN5V2pG....5VVtN5.private and lies:
```
cd $HOME/.transformers/cert
```

