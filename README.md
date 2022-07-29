## <p align=center> _Solace Protocol_ for NEAR </p>

![image](https://user-images.githubusercontent.com/103751566/181745625-12b68f87-e20e-4b85-b081-f774e66e8d2d.png)


Solace is a smart-contract based non-custodial wallet for Near which eases user's onboarding and enhances security using social recovery, written in Rust.

### The Problem

Seed Phrases deter mass adoption of crypto wallets. Users tend to lose their seed phrases or they are a vector of attack by hackers. Having a single point of failure for user's funds, was never a good thing.

Social recovery based smart-contract wallets are the way of the future. They remove barrier to adoption by making seed-phrases a thing of the past and provide multiple layers of security to the user's funds.

### How does it work

User creates a Solace Smart-Contract Wallet Account, signed by a Keypair generated for them
This signing keypair becomes the owner of the Smart Contract Wallet and is required for signing any transaction
The user then adds guardians to guard the Program Wallet. These guardians can be friends / family or other wallets the user has access to (Phantom, Ledger, ...)
In case of losing access to the signing key pair (device theft, damage, hack, ...), the user can initiate the recovery process for their existing Program Wallet with a new Signing Keypair, and request the guardians to approve the request on-chain
If the guardians approve of this over a threshold (3/5 or 5/7 majority as set initially), the program wallet's new owner now becomes keypair requesting for recovery.
Thus, the user's funds were never jeopardized as a result of losing the device or the seed phrase as a recovery request freezes any funds from moving out
A timelock is used to prevent the hacker / malicious user from changing the guardian, such that a recovery is always prioritized over changing a guardian

### Advantages

Ease of Onboarding - Seed phrase to the user's signing account is redundant, and not as important as it would've been if it held all of the user's assets
Fund Security Funds are stored in the smart-contract, and is protected by the recovery_mode flag, which prevents funds from leaving the system

### Protocol

Solace is a wallet protocol, which allows anyone to build their UI Layer on top of it. We have built the Solace Wallet App and the Solace Backend to demonstrate the capabilities of Solace

===================

# Prerequisites

If you're using Gitpod, you can skip this step.

1. Make sure Rust is installed per the prerequisites in [`near-sdk-rs`](https://github.com/near/near-sdk-rs#pre-requisites)
2. Ensure `near-cli` is installed by running `near --version`. If not installed, install with: `npm install -g near-cli`

## Building

To build run:

```bash
cd scripts && ./build.sh
```

# Using this contract

### Quickest deploy

You can build and deploy this smart contract to a development account. [Dev Accounts](https://docs.near.org/concepts/basics/account#dev-accounts) are auto-generated accounts to assist in developing and testing smart contracts. Please see the [Standard deploy](#standard-deploy) section for creating a more personalized account to deploy to.

```bash
near dev-deploy --wasmFile res/fungible_token.wasm --helperUrl https://near-contract-helper.onrender.com
```

Behind the scenes, this is creating an account and deploying a contract to it. On the console, notice a message like:

> Done deploying to dev-1234567890123

In this instance, the account is `dev-1234567890123`. A file has been created containing a key pair to
the account, located at `neardev/dev-account`. To make the next few steps easier, we're going to set an
environment variable containing this development account id and use that when copy/pasting commands.
Run this command to the environment variable:

```bash
source neardev/dev-account.env
```

You can tell if the environment variable is set correctly if your command line prints the account name after this command:

```bash
echo $CONTRACT_NAME
```

The next command will initialize the contract using the `new` method:

```bash
near call $CONTRACT_NAME new '{"owner_id": "'$CONTRACT_NAME'", "total_supply": "1000000000000000", "metadata": { "spec": "ft-1.0.0", "name": "Example Token Name", "symbol": "EXLT", "decimals": 8 }}' --accountId $CONTRACT_NAME
```

To get the fungible token metadata:

```bash
near view $CONTRACT_NAME ft_metadata
```

### Standard deploy

This smart contract will get deployed to your NEAR account. For this example, please create a new NEAR account. Because NEAR allows the ability to upgrade contracts on the same account, initialization functions must be cleared. If you'd like to run this example on a NEAR account that has had prior contracts deployed, please use the `near-cli` command `near delete`, and then recreate it in Wallet. To create (or recreate) an account, please follow the directions on [NEAR Wallet](https://wallet.near.org/).

Switch to `mainnet`. You can skip this step to use `testnet` as a default network.

    export NEAR_ENV=mainnet

In the project root, log in to your newly created account with `near-cli` by following the instructions after this command:

    near login

To make this tutorial easier to copy/paste, we're going to set an environment variable for your account id. In the below command, replace `MY_ACCOUNT_NAME` with the account name you just logged in with, including the `.near`:

    ID=MY_ACCOUNT_NAME

You can tell if the environment variable is set correctly if your command line prints the account name after this command:

    echo $ID

Now we can deploy the compiled contract in this example to your account:

    near deploy --wasmFile res/fungible_token.wasm --accountId $ID

FT contract should be initialized before usage. You can read more about metadata at ['nomicon.io'](https://nomicon.io/Standards/FungibleToken/Metadata.html#reference-level-explanation). Modify the parameters and create a token:

    near call $ID new '{"owner_id": "'$ID'", "total_supply": "1000000000000000", "metadata": { "spec": "ft-1.0.0", "name": "Example Token Name", "symbol": "EXLT", "decimals": 8 }}' --accountId $ID

Get metadata:

    near view $ID ft_metadata

## Transfer Example

Let's set up an account to transfer some tokens to. These account will be a sub-account of the NEAR account you logged in with.

    near create-account bob.$ID --masterAccount $ID --initialBalance 1

Add storage deposit for Bob's account:

    near call $ID storage_deposit '' --accountId bob.$ID --amount 0.00125

Check balance of Bob's account, it should be `0` for now:

    near view $ID ft_balance_of '{"account_id": "'bob.$ID'"}'

Transfer tokens to Bob from the contract that minted these fungible tokens, exactly 1 yoctoNEAR of deposit should be attached:

    near call $ID ft_transfer '{"receiver_id": "'bob.$ID'", "amount": "19"}' --accountId $ID --amount 0.000000000000000000000001

Check the balance of Bob again with the command from before and it will now return `19`.
