# How to run a validator node on Near Protocol

## Setup enviroment

### First, let's make sure the linux machine is up-to-date.
```bash
sudo apt update && sudo apt upgrade -y
```

### Install developer tools, Node.js, and npm

#### Install developer tools:
```bash
sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
```
#####  Install Python pip:

```bash
sudo apt install python3-pip
```
##### Set the configuration:

```bash
USER_BASE_BIN=$(python3 -m site --user-base)/bin
export PATH="$USER_BASE_BIN:$PATH"
```

##### Install Building env
```bash
sudo apt install clang build-essential make
```

##### Install Rust & Cargo
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

##### Source the environment
```
source $HOME/.cargo/env
```

#### Install `Node.js` and `npm`:
```bash
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
sudo apt install build-essential nodejs
PATH="$PATH"
```

##### Check `Node.js` and `npm` version:
```bash
node -v
```
> v18.x.x

```bash
npm -v
```
> 8.x.x

##### Install NEAR-CLI
Here's the Github Repository for NEAR CLI.: https://github.com/near/near-cli. To install NEAR-CLI, unless you are logged in as root, which is not recommended you will need to use `sudo` to install NEAR-CLI so that the near binary is saved to /usr/local/bin

```bash
sudo npm install -g near-cli
```

Now that NEAR-CLI is installed, let's test out the CLI and use the following commands to interact with the blockchain as well as to view validator stats. There are three reports used to monitor validator status:


##### Environment
The environment will need to be set each time a new shell is launched to select the correct network.

Networks:
- GuildNet
- TestNet
- MainNet
- **Shardnet** (this is the network we will use for Stake Wars)

Command:
```bash
export NEAR_ENV=shardnet
```

You can also run this command to set the Near testnet Environment persistent:
```bash
echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
```

## Create a wallet and deploy staking pool contract

### Create a wallet on shardnet

Go to https://wallet.shardnet.near.org/ and create a wallet. After create wallet, you will be received 2000 Near on shardnet for run a validator node.

### Authorize Wallet Locally

A full access key needs to be installed locally to be able to sign transactions via NEAR-CLI.


* You need to run this command:

```bash
near login
```

> Note: This command launches a web browser allowing for the authorization of a full access key to be copied locally.

1 – Copy the link in your browser


![img](https://github.com/near/stakewars-iii/blob/main/challenges/images/1.png)

2 – Grant Access to Near CLI

![img](https://github.com/near/stakewars-iii/blob/main/challenges/images/3.png)

3 – After Grant, you will see a page like this, go back to console

![img](https://github.com/near/stakewars-iii/blob/main/challenges/images/4.png)

4 – Enter your wallet and press Enter

![img](https://github.com/near/stakewars-iii/blob/main/challenges/images/5.png)

### Generate staking pool public key

Choose your pool id, such as: pool-1. Then your pool account id will be pool-1.factory.shardnet.near

Command:
```bash
near generate-key pool-1.factory.shardnet.near
```

> Key pair with ed25519:BMmyrZdJ97CsjncYki1WPCU21v5maCFtnS6s2gBe56G9 public key for an account "pool-1.factory.shardnet.near"

Remember to copy this public key.

### Deploy staking pool contract

Calls the staking pool factory, creates a new staking pool with the specified name, and deploys it to the indicated accountId.

```bash
near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "pool-1", "owner_id": "<your wallet account id>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<your wallet account id>" --amount=30 --gas=300000000000000
```

From the example above, you need to replace:

* **Pool ID**: Staking pool name, the factory automatically adds its name to this parameter, creating {pool_id}.{staking_pool_factory}
Examples:   

- If pool id is stakewars will create : `pool-1.factory.shardnet.near`

* **Owner ID**: The SHARDNET account (i.e. stakewares.shardnet.near) that will manage the staking pool.
* **Public Key**: The public key you have generated.
* **5**: The fee the pool will charge (e.g. in this case 5 over 100 is 5% of fees).
* **Account Id**: The SHARDNET account deploying and signing the mount tx.  Usually the same as the Owner ID.

> Be sure to have at least 30 NEAR available, it is the minimum required for storage.

#### Transactions Guide
##### Deposit and Stake NEAR

Command:
```bash
near call <staking_pool_id> deposit_and_stake --amount <amount> --accountId <accountId> --gas=300000000000000
```
##### Unstake NEAR
Amount in yoctoNEAR.

Run the following command to unstake:
```bash
near call <staking_pool_id> unstake '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
To unstake all you can run this one:
```bash
near call <staking_pool_id> unstake_all --accountId <accountId> --gas=300000000000000
```
##### Withdraw

Unstaking takes 2-3 epochs to complete, after that period you can withdraw in YoctoNEAR from pool.

Command:
```bash
near call <staking_pool_id> withdraw '{"amount": "<amount yoctoNEAR>"}' --accountId <accountId> --gas=300000000000000
```
Command to withdraw all:
```bash
near call <staking_pool_id> withdraw_all --accountId <accountId> --gas=300000000000000
```

##### Ping
A ping issues a new proposal and updates the staking balances for your delegators. A ping should be issued each epoch to keep reported rewards current.

Command:
```bash
near call <staking_pool_id> ping '{}' --accountId <accountId> --gas=300000000000000
```
Balances
Total Balance
Command:
```bash
near view <staking_pool_id> get_account_total_balance '{"account_id": "<accountId>"}'
```
##### Staked Balance
Command:
```bash
near view <staking_pool_id> get_account_staked_balance '{"account_id": "<accountId>"}'
```
##### Unstaked Balance
Command:
```bash
near view <staking_pool_id> get_account_unstaked_balance '{"account_id": "<accountId>"}'
```
##### Available for Withdrawal
You can only withdraw funds from a contract if they are unlocked.

Command:
```bash
near view <staking_pool_id> is_account_unstaked_balance_available '{"account_id": "<accountId>"}'
```
##### Pause / Resume Staking
###### Pause
Command:
```bash
near call <staking_pool_id> pause_staking '{}' --accountId <accountId>
```
###### Resume
Command:
```bash
near call <staking_pool_id> resume_staking '{}' --accountId <accountId>
```

## Activating the node as validator

### Build neard binary

#### Clone `nearcore` project from GitHub
First, clone the [`nearcore` repository](https://github.com/near/nearcore).

```bash
git clone https://github.com/near/nearcore
cd nearcore
git fetch
```

Checkout to the commit needed. Please refer to the commit defined in [this file](commit.md). 
```
git checkout <commit>
```

#### Compile `nearcore` binary
In the `nearcore` folder run the following commands:

```bash
cargo build -p neard --release --features shardnet
```
The binary path is `target/release/neard`. If you are seeing issues, it is possible that cargo command is not found. Compiling `nearcore` binary may take a little while.

#### Initialize working directory

In order to work properly, the NEAR node requires a working directory and a couple of configuration files. Generate the initial required working directory by running:

```bash
./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis
```

![img](https://github.com/near/stakewars-iii/blob/main/challenges/images/initialize.png?raw=true)

This command will create the directory structure and will generate `config.json`, `node_key.json`, and `genesis.json` on the network you have passed. 

- `config.json` - Configuration parameters which are responsive for how the node will work. The config.json contains needed information for a node to run on the network, how to communicate with peers, and how to reach consensus. Although some options are configurable. In general validators have opted to use the default config.json provided.

- `genesis.json` - A file with all the data the network started with at genesis. This contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain. The genesis.json file is a snapshot of the network state at a point in time. In contacts accounts, balances, active validators, and other information about the network. 

- `node_key.json` -  A file which contains a public and private key for the node. Also includes an optional `account_id` parameter which is required to run a validator node (not covered in this doc).

- `data/` -  A folder in which a NEAR node will write it's state.

#### Replace the `config.json`

From the generated `config.json`, there two parameters to modify:
- `boot_nodes`: If you had not specify the boot nodes to use during init in Step 3, the generated `config.json` shows an empty array, so we will need to replace it with a full one specifying the boot nodes.
- `tracked_shards`: In the generated `config.json`, this field is an empty empty. You will have to replace it to `"tracked_shards": [0]`

```bash
rm ~/.near/config.json
wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
```

#### Get latest snapshot

**IMPORTANT: NOT REQUIRED TO GET SNAPSHOT AFTER HARDFORK ON SHARDNET DURING 2022-07-18**

Install AWS Cli
```bash
sudo apt-get install awscli -y
```

Download snapshot (genesis.json)

** IMPORTANT: NOT REQUIRED TO GET SNAPSHOT AFTER HARDFORK ON SHARDNET DURING 2022-07-18**

```bash
cd ~/.near
wget https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json
```

If the above fails, AWS CLI may be oudated in your distribution repository. Instead, try:
```bash
pip3 install awscli --upgrade
```

* Copy the file generated to shardnet folder:
Make sure to replace <pool_id> by your accountId
```bash
cp ~/.near-credentials/shardnet/pool-1.factory.shardnet.near.json ~/.near/validator_key.json
```

* Change `private_key` to `secret_key`

> Note: The account_id must match the staking pool contract name or you will not be able to sign blocks.\

File content must be in the following pattern:
```
{
  "account_id": "xx.factory.shardnet.near",
  "public_key": "ed25519:HeaBJ3xLgvZacQWmEctTeUqyfSU4SDEnEwckWxd92W2G",
  "secret_key": "ed25519:****"
}
```
* Setup Systemd
Command:

```bash
sudo vi /etc/systemd/system/neard.service
```
Paste:

```
[Unit]
Description=NEARd Daemon Service

[Service]
Type=simple
User=<USER>
#Group=near
WorkingDirectory=/home/<USER>/.near
ExecStart=/home/<USER>/nearcore/target/release/neard run
Restart=on-failure
RestartSec=30
KillSignal=SIGINT
TimeoutStopSec=45
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

> Note: Change USER to your paths

Command:

```bash
sudo systemctl enable neard
```
Command:

```bash
sudo systemctl start neard
```
If you need to make a change to service because of an error in the file. It has to be reloaded:

```bash
sudo systemctl reload neard
```
###### Watch logs
Command:

```bash
journalctl -n 100 -f -u neard
```
Make log output in pretty print

Command:

```bash
sudo apt install ccze
```
View Logs with color

Command:

```bash
journalctl -n 100 -f -u neard | ccze -A
```
#### Becoming a Validator
In order to become a validator and enter the validator set, a minimum set of success criteria must be met.

* The node must be fully synced
* The `validator_key.json` must be in place
* The contract must be initialized with the public_key in `validator_key.json`
* The account_id must be set to the staking pool contract id
* There must be enough delegations to meet the minimum seat price. See the seat price [here](https://explorer.shardnet.near.org/nodes/validators).
* A proposal must be submitted by pinging the contract
* Once a proposal is accepted a validator must wait 2-3 epoch to enter the validator set
* Once in the validator set the validator must produce great than 90% of assigned blocks

Check running status of validator node. If “Validator” is showing up, your pool is selected in the current validators list.
