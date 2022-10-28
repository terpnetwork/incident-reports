# Terp-Core October 23rd, 2022 Chain Halt Upgrade

This upgrade is effectively a hard fork with a different chain ID. This means we'll need to delete all previous data and start from a new genesis.

NOTE: This assumes you've already ran through setting up a terp-core node here: https://docs.terp.network/validators/getting-setup.html

## Performing the Upgrade

### 1. **Stop the node**
```
sudo systemctl stop terpd.service
```
### 2. **Upgrade go**
```
wget https://go.dev/dl/go1.19.2.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz
``` 


### 3. **Backup priv_validator_key.json**
This is only relevant if you're upgrading your validator. Your priv_validator_key.json is how your validator is identified. If you haven't backed it up already, **DO SO NOW**.

An example method for doing so is as follows, which will copy your validator key to the home dir:
```sh
cp ~/.terp/config/priv_validator_key.json ~/
diff -s ~/.terp/config/priv_validator_key.json ~/priv_validator_key.json
```

> Should return "Files ... and ... are identical" if not, make sure to manually backup your priv_validator_state.json file.

*NOTE*: keeping your copy of the validator key on the same machine is NOT sufficient. At a bare minimum it should be backed up locally so you always have access to it.

### 4. **Purge previous chain state and addrbook.json**
Because we'll be starting from a new genesis, the previous data is no longer necessary.
```sh
terpd tendermint unsafe-reset-all --home $HOME/.terp 
```

### 5. **Move Private Validator State**
```
cp ~/priv_validator_state.json ~/.terp/data/priv_validator_state.json
diff -s ~/.terp/data/priv_validator_state.json ~/priv_validator_state.json
```
> Should return "Files ... and ... are identical"



### 6. **Purge current peers and seeds from config.toml**
This is done to ensure all peers are clean when moving forward. The addrbook.json file was already purged in the previous step.

**WARNING:** DO NOT DO BLANK PERSISTENT PEERS IF YOU ARE RUNNING SENTRIES. Only remove persistent peers that are not your sentries/val node.

```sh
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"\"/" ~/.terp/config/config.toml
sed -i.bak -e "s/^seeds *=.*/seeds = \"\"/" ~/.terp/config/config.toml
```

### 7. **Add seeds and peers to config.toml**
These are all verified to be using the new genesis file and binary.

**WARNING:** These should be added manually if you are running a sentries setup, or you will blank out your peers.
```sh
SEEDS=""
PEERS=""
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" ~/.terp/config/config.toml
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.terp/config/config.toml
```

### 8. **Download and install the new binary**

#### 8a. Install GO v19.2
wget https://go.dev/dl/go1.19.2.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.19.2.linux-amd64.tar.gz

#### 8b. Upgrade or Install Terp-Core
`Update`
```
cd ~/terp-core
git fetch --all
git checkout v0.1.2
make install
terpd version
```
`Install`
```
git clone https://github.com/terpnetwork/terp-core.git
cd terp-core
git fetch --all
git checkout v0.1.2
make install
terpd version
```
#### 8c. **[OPTIONAL] If you use cosmovisor**
You will need to re-setup cosmovisor with the new genesis.
```sh
rm $DAEMON_HOME/cosmovisor/genesis/bin/terpd
rm -rf $DAEMON_HOME/cosmovisor/upgrades
mkdir $DAEMON_HOME/cosmovisor/upgrades
cp $HOME/go/bin/terpd $DAEMON_HOME/cosmovisor/genesis/bin
rm $DAEMON_HOME/cosmovisor/current
```

Check terpd has copied to the new location.
```sh
$DAEMON_HOME/cosmovisor/genesis/bin/terpd version

# returns
v0.1.2

tree $DAEMON_HOME/cosmovisor

# returns
$HOME.terp/cosmovisor
├── genesis
│   └── bin
│       └── terpd
└── upgrades
```

### 9. **Download the Alberta genesis**

To build yourself or check options, [read more here](./genesis.md).

```sh
cd ~/.terp/config/genesis.json
rm ~/.terp/config/genesis.json
wget https://raw.githubusercontent.com/terpnetwork/test-net/master/athena-2/genesis.json
mv genesis.json $HOME/.terp/config/genesis.json

# check chain is athena-2, genesis time is correct & initial block is 694200
# note if using zsh that you may need to break this up, and run steps individually
# i.e. cat $HOME/.terp/config/genesis.json | jq '.chain_id'
cat $HOME/.terp/config/genesis.json | jq '"Genesis Time: " + .genesis_time + " — Chain ID: " + .chain_id + " - Initial Height: " + .initial_height'
```

### 9. **Verify genesis shasum**

```sh
jq -S -c -M '' ~/.terp/config/genesis.json | sha256sum

# this will return
# b2acc7ba63b05f5653578b05fc5322920635b35a19691dbafd41ef6374b1bc9a  /home/user/.terp/config/genesis.json 
```

### 10. **Be paranoid**
This isn't strictly necessary - you can skip it. However, you might want to be paranoid and just double-check.
```sh
terpd tendermint unsafe-reset-all --home $HOME/.terp
```

### 11. **Restore priv_validator_key.json and priv_validator_state.json**

**Important** By resetting the state we also resetted the validator sign state, which may cause double sign. We need to restore our backup to prevent this. 

If you are using a remote signer this step is probably not needed

```sh
cp ~/priv_validator_state.json ~/.terp/data/priv_validator_state.json
cp ~/priv_validator_key.json ~/.terp/config/priv_validator_key.json
```

### 12. **Start the node**
```sh
sudo systemctl restart terpd
```

### 13. **Confirm the process running**
```sh
sudo journalctl -fu terpd
```

the output should be as follows:
```
4:58AM INF Genesis time is in the future. Sleeping until then... genTime=TBD
```

---

## FAQ

### What about ibc transactions?
They will time out and be returned to your wallets. 

### Do I need to resync?
Nope! We are effectively starting from "block 0" of the new genesis. Similarly, all snapshots no longer matter (except archive snapshots, for data integrity purposes). This is the perfect time to create a backup node or sentries if you didn't have them previously.

### What about staking rewards and/or commission?
These will be auto-claimed for you. Therefore, they are safe!

### How was the new genesis created?
The current state of athena-1 was exported from block `694200`, then converted into a genesis file. Essentially that means all previously state was backed up as-is. 
