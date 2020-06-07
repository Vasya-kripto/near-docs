# Introduction

This document contains instructions for compiling the nearcore code from beta branch and running a validator node using the compiled code. Running a node using this method would be a requirement for Stakewars II. The steps provided in this document are tested on a Linux Ubuntu server (18.04) but may also work for other Linux versions.

To join stakewars you need to:

- Create Betanet Account at https://wallet.betanet.near.org/
- Create a Linux VPS (I am using a Standard B2s (2 vcpus, 4 GiB memory) + 128 GB hard disk on Azure with Ubuntu 18.04). Check out the hardware requirements [here](https://docs.near.org/docs/roles/validator/hardware).
- Pull and Compile [nearcore](https://github.com/nearprotocol/nearcore.git) beta branch
- Run nearup using the compiled nearcore client
- Deploy your staking pool using [Staking Pool Factory](https://github.com/near/initial-contracts/tree/master/staking-pool-factory)

This document focuses on 

- compiling the nearcore client from beta branch and 
- running validator node using nearup using the compiled code
- creating and deploying and staking pool contract using Staking Pool Factory UI

Please read the instructions on https://github.com/nearprotocol/stakewars for enrolling for Stakewars II and getting betanet tokens for staking.

This document assumes that you are already familar with using near-shell and have it configured on your local machine. It also assumes that you are familiar with process of authorising access from near-shell using `near login`. If not, please setup near-shell and get yourself familiarised with basic commands from [Near Docs](https://docs.near.org/docs/development/near-shell#__docusaurus). 

### Install nearup
The instructions for installing and running nearup are avaialble at https://github.com/near/nearup. Follow the steps at the provided link for installing nearup. **Do not start the nearup yet**. We will run nearup using compiled nearcore client.


### Clone nearcore Beta Branch
Stakewars II uses the nearcore beta branch for the validator node. It is recommended to clone the code in a folder identified by latest commit hash (for example if the latest build hash is 74bfab78 then in the git command use folder name as nearcore-74bfab78). It makes easier to revert between the versions of nearcore quickly if there are issues with a particular release. To keep the nearcore versions separate (there is a weekly release of updated code in beta branch) you can follow these steps:


```
git clone -b beta https://github.com/nearprotocol/nearcore.git nearcore-74bfab78
```

After the code is successfully cloned use the `cd` command to navigate to the latest directory. The folder name provided below is just for reference. You will have to replace it with the correct name it's created with.

```
cd nearcore-74bfab78
```

Once in the folder, verify that you are actually on beta branch

```
git branch
```

If you see something lile `*beta` in the output it means you have correctly checked out beta branch.

### Build nearcore

You can build nearcore in couple of different ways. You can either choose to do a full build using `make release` or you can simply run the following command which only runs a single cargo build for nearcore/neard.

```
cargo build -p neard --release
```

This command, after successful completion, creates the nearcore client binanries in `nearcore/target/release` directory.


The limited build above may save some build time as it avoids doing following additional builds if you use `make release` (please refer to [Makefile](https://github.com/nearprotocol/nearcore/blob/beta/Makefile) in nearcore).

```
	cargo build -p keypair-generator --release
	cargo build -p genesis-csv-to-json --release
	cargo build -p near-vm-runner-standalone --release
	cargo build -p state-viewer --release
	cargo build -p store-validator --release
```

### Run nearup Using Compiled nearcore Client
Now you can start the validator node on betanet using the following command (change the nearcore folder name according to which version you want to run. nearcore-613953cf is used just for reference in the command below).

```
nearup betanet --nodocker --binary-path $HOME/nearcore-74bfab78/target/release
```
When you run the nearup command for the very first time it asks for an account ID. You will see a message similar to following on command line

```
Enter your account ID (leave empty if not going to be a validator):
```
Enter the ID which you will use for creating and deploying a staking pool contract in next step. E.g. I used the name of my staking pool contract i.e. *mutedtommy-staking-pool.stakehouse.betanet* (to be created and deployed in next section). 

Don't forget to add *.stakehouse.betanet* to the ID. This is required if you want to use the Staking Pool Contract Factory UI for generating and deploying the staking pool contract. Notedown this ID as we will be using this to generate and deploy staking pool contract in next section.

After you have provided the account ID and hit enter the node will start and you will see messages similar to following on command line

```
Generating node key...
PK: ed25519:ErcckoexbhyuJcYkD4cHwhuBtVMnHaaakjJyc5c5abcd
Node key generated
Generating validator key...
PK: ed25519:4XZfr69xBCYaws89y83kAKTeSt1238z3G2s6qL
Validator key generated
Stake for user 'xxxxxxx' with 'ed25519:4XZfr69xBCYaws89y83kAKTeSt1238z3G2s6qL'
Starting NEAR client...
Node is running! 
To check logs call: `nearup logs` or `nearup logs --follow`

```

You can check the logs to see if nearup started without any errors by using `nearup logs -f`


## Deploy a Staking Pool using Staking Pool Factory
You can use the Staking Pool Factory UI to deployig a staking pool. Open https://near-examples.github.io/staking-pool-factory/ and login using your betanet wallet. On the Staking Pool Factory UI you will see the following fields:

##### Staking Pool ID
In this field enter the same ID you used while starting nearup in previous step

**this is very important that you are entering the same id which you used while starting the nearup**

##### Owner Id
Enter the wallet ID of the account which will collect fee for this staking pool (you can use the betanet wallet you created for Stakewars II)

##### Initial Staking Public Key

Enter the public key which was generated by nearup at the time of start up. You can get the the key from *public key* field from the output of `more ~/.near/betanet/validator_key.json | grep 'public_key\|account_id'` on the server (sample output provided below - **only use the value after ed25519:**)

```
  "account_id": "mutedtommy-staking-pool.stakehouse.betanet",
  "public_key": "ed25519:ABCDErY50LawhGZ11Hcfu4JtrUTmZr765RTHFUoibd6"
```

##### Initial Reward Fee Fraction
This is %age of fee you will be collecting on generated rewards for running this stake pool. Enter the amount you want to charge as fee.

Once you have entered and double checked the values click on "Create Staking Pool" button. It requires at least 30 Near in your account to initiate this transaction. If the contract is successfully created you will see a message like following towards the top of the page.

*Successfully created your staking pool @mutedtommy-staking-pool-2.stakehouse.betanet*

Your newly created stake pool is avaialble for deposits and staking now. If your staking pool has enough NEAR to get a seat as a validator it will appear as a valiadtors in approximately 6 hours (2 epochs). You can get an idea about approximate number of NEAR required to get a set as a validator by running the following command:

```
near validators next | grep "seat price"
```

Make sure the near-shell on your computer is using betanet. You can make the near-shell use betanet by:

```
export NODE_ENV=betanet
```

## Depositing and Staking to the Staking Pool

You can deposit and stake to the newly deployed contract using following commands.

```
near call <name of the stake pool you created earlier> deposit '{}' --accountId <wallet from where you want to deposit> --amount <amount you want to deposit>
```
Following is an example of how I deposited near to my staking pool from my near betanet wallet.

```
near call mutedtommy-staking-pool-2.stakehouse.betanet deposit '{}' --accountId mutedtommy.betanet --amount 3000
```

If the deposit was successfull you will see a message similar to following.

```
Scheduling a call: mutedtommy-staking-pool-2.stakehouse.betanet.deposit({}) with attached 3000 NEAR
[mutedtommy-staking-pool-2.stakehouse.betanet]: @mutedtommy.betanet deposited 3000000000000000000000000000. New unstaked balance is 3000000000000000000000000000
''
```

You can now stake the deposited amount using the following command

```
near call <name of the stake pool you deposited near token to> stake '{"amount": "<amount you want to stake>"}' --accountId <wallet id from where you made the deposit>
```

I used the following command to stake to my staking pool.

```
near call mutedtommy-staking-pool-2.stakehouse.betanet stake '{"amount": "3000000000000000000000000000"}' --accountId mutedtommy.betanet
```

If the staking call is successful you will see a message similar to following.

```
Scheduling a call: mutedtommy-staking-pool-2.stakehouse.betanet.stake({"amount": "3000000000000000000000000000"})
[mutedtommy-staking-pool-2.stakehouse.betanet]: @mutedtommy.betanet staking 3000000000000000000000000000. Received 2999999996874265166646757757 new staking shares. Total 0 unstaked balance and 2999999996874390698166169529 staking shares
[mutedtommy-staking-pool-2.stakehouse.betanet]: Contract total staked balance is 3030000000001254315194165001. Total number of shares 3029999996874389698166169529
''

```

## My Staking Pool Details
I am running @mutedtommy-staking-pool-2.stakehouse.betanet on betanet using these steps. If you wish to delegate to my staking pool please use the steps mentioned in previous section (Depositing and Staking to the Staking Pool).

Any suggestions to improve the document are welcome.
