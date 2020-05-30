# Introduction

This document contains instructions for compiling the nearcore code from beta branch and running a validator node using the compiled code. Running a node using this method would be a requirement for Stakewars II. The information provided in this document is meant to be followed on a Linux Ubuntu server but it may also work for other Linux versions.

To join stakewars you need to:

- Create Betanet Account at https://wallet.betanet.near.org/
- Create a Linux Ubuntu/Debian VPS (I am using a Standard B2s (2 vcpus, 4 GiB memory) + 128 GB hard disk on Azure with Ubuntu 18.04)
- Pull and Compile [nearcore](https://github.com/nearprotocol/nearcore.git) beta branch
- Run nearup using the compiled nearcore client
- Deploy your staking pool using [Staking Pool Factory](https://github.com/near/initial-contracts/tree/master/staking-pool-factory)

This document focuses only on 

- compiling the nearcore client from beta branch and 
- running validator node using nearup using the compiled code 

I will create another document which will provide details of deploying a staking pool contract using Stake Pool Factory.

### Install nearup
The instructions for installing and running nearup are avaialble at https://github.com/near/nearup. Follow the steps at the provided link for installing nearup. **Do not start the nearup yet**. We will run nearup using compiled nearcore client.


### Clone nearcore Beta Branch
Stakewars II uses the nearcore beta branch for the validator node. You can clone the nearcore repo and checkout beta branch using following git command on server CLI. 

```
git clone -branch beta https://github.com/nearprotocol/nearcore.git

```
After cloning the repo execute following command from CLI to move to nearcore directory for build

```
cd nearcore
```

Verify that you are actually on beta branch


```
git branch
```

If you see something lile `*beta` in the output it means you have correctly checked out beta branch.

### Build nearcore
After succefull cloning the repo and making sure you working with beta branch run the following command to start the compilation.

```
make release
```
This command, after successful completion, creates the nearcore client binanries in `nearcore/target/release` directory.

### Run nearup Using Compiled nearcore Client
Now you can start the validator node on betanet using the following command

```
nearup betanet --nodocker --binary-path $HOME/nearcore/target/release
```

