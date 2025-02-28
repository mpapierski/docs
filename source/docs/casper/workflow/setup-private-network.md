# Set Up a Private Casper Network


Casper private networks operate in a similar way to the Casper public network. The significant difference in private networks is a closed validator set and having administrator account(s) which can control regular accounts. Hence, there are specific configurations when setting up the genesis block and administrator accounts. Besides the main configuration options that the Casper platform provides, each customer may add other configuration options when setting up a private network.


## Prerequisites
Follow these guides to set up the required environment and user accounts. 
- [Setting up the Casper client](/workflow/setup/#the-casper-command-line-client)
- [Setting up the client for interacting with the network](https://github.com/casper-network/casper-node/blob/master/client/README.md#casper-client)
- [Setting up an account](/workflow/setup/#setting-up-an-account)


## Step 1. Setting up a Validator Node
A [Casper node](/glossary/N/#node) is a physical or virtual device participating in the Casper network. You need to set up several [validator](/glossary/V/#validator) nodes on your private network. An [operator](/glossary/O/#operator) who has won an [auction](/glossary/A/#auction) bid will be a validator for the private network.


Use the below guides to set up and manage validator nodes.

- [Casper node setup - GitHub guide](https://github.com/casper-network/casper-node/tree/master/resources/production#casper-node-setup): A guide to configuring a system with the new Rust node to operate within a network.
- [Basic node setup tutorial](/operators/setup/): A guide on using the `casper-node-launcher`, generating directories and files needed for running casper-node versions and performing upgrades, generating keys, and setting up the configuration file for nodes.
- [Set up Mainnet and Testnet validator nodes](https://docs.cspr.community/): A set of guides for Mainnet and Testnet node-operators on setting up and configuring their Casper network validator nodes.

Use these FAQ collections for tips and details for validators.
- [FAQs for basic validator node ](https://docs.casperlabs.io/faq/faq-validator/)
- [FAQs on Main Net and Test Net validator node setup](https://docs.cspr.community/docs/faq-validator.html)

## Step 2. Setting up the Directory
Use these guides to set up your private network directories. You will find several main directories dedicated to different purposes.

- Go through the [file location](/operators/setup/#file-locations) section to understand how directories are created and managed in a Casper private network. 
- Refer to the [setting up a new network](/operators/create/) guide to identify the required configuration files to set up a genesis block.

## Step 3. Configuring the Genesis Block
A Casper private network contains a different set of configurations when compared to the public network. The `chainspec.toml` file contains the required configurations for the genesis process in a private network. 

You should add the configuration options below to the `chainspec.toml` file inside the [private network directory](/workflow/setup-private-network/#step-2-setting-up-the-directory).

### Unrestricted transfers configuration
This option disables unrestricted transfers between regular accounts. A regular account user cannot do a fund transfer when this attribute is set to false. Only administrators can transfer tokens freely between users and other administrators. 

```toml
[core]
allow_unrestricted_transfers = false
```
In contrast, users in the public network can freely transfer funds to different accounts.

:::note
A Casper private network doesn't support the minting process. Only admininstrator accounts can maintain funds. This is enabled by configuring these options:

```toml
[core]
allow_unrestricted_transfers = false
compute_rewards = false
allow_auction_bids = false
refund_handling = { type = "refund", refund_ratio = [1, 1] }
fee_handling = { type = "accumulate" }
administrators = ["ADMIN_PUBLIC_KEY"]
```
:::

### Refund handling configuration
This option manages the refund behavior at the finalization of a deploy execution. It changes the way the Wasm execution fees are distributed. After each deploy execution, the network calculates the amount of gas spent for the execution and manages to refund any remaining tokens to the user.

A `refund_ratio` is specified as a proper fraction (the numerator must be lower or equal to the denominator). In the example below, the `refund_ratio` is 1:1. If 2.5 CSPR is paid upfront and the gas fee is 1 CSPR, 1.5 CSPR will be given back to the user.


```toml
[core]
refund_handling = { type = "refund", refund_ratio = [1, 1] } 
```
After deducting the gas fee, the distribution of the remaining payment amount is handled based on the [fee_handling](/workflow/setup-private-network/#fee-handling-config) configuration.

The default configuration for a public chain, including the Casper Mainet,  looks like this:

```toml
[core]
refund_handling = { type = "refund", refund_ratio = [0, 100] }
```
The refund variant with `refund_ratio` of [0, 100] means that 0% is given back to the user after deducting gas fees. In other words, if a user paid 2.5 CSPR and the gas fee is 1 CSPR, the user will not get the remaining 1.5 CSPR in return.

### Fee handling configuration
This option defines how to distribute the fees after refunds are handled. While refund handling defines the amount we pay back after a transaction, fee handling defines the methods of fee distribution after a refund is performed.


Set up the configuration as below:

```toml
[core]
fee_handling = { type = "pay_to_proposer" }
```

The `fee_handling` configuration has three variations:
- `pay_to_proposer`: The rest of the payment amount after deducing the gas fee from a refund is paid to the block's [proposer](/glossary/P/#proposer).
- `burn`: The tokens paid are burned, and the total supply is reduced.
- `accumulate`: The funds are transferred to a special accumulation purse. Here, the accumulation purse is owned by a handle payment system contract, and the amount is distributed among all the administrators defined at the end of a switch block. The fees are paid to the purse owned by the handle payment contract, and no tokens are transferred to the proposer when this configuration is enabled.

### Auction behavior configuration

A private network requires to have a fixed set of validators. This configuration restricts the addition of new validators to the private network. Hence, you are not allowed to bid new entries into the validator set.


Use the configuration below to limit the auction validators:


```toml
[core]
allow_auction_bids = false
```

Other configurations related to the auction:

- `allow_auction_bids` - if this option is set to *false* then `add_bid` and `delegate` options are disabled. It also disables adding new validators to the system. Invoking those entry points leads to an `AuctionBidsDisabled` error.
- `core.compute_rewards` - if this option is set to *false*, then all the rewards on a switch block will be set to 0. The auction contract wouldn't process rewards distribution that would increase validator bids.

In a public network, `allow_auction_bid` is set to *true*, which allows bidding for new entries and validator nodes. 

## Step 4. Configuring the Administrator Accounts
An administrator is mandatory for a private network since it manages all the other [validator](/glossary/V/#validator) accounts. There should be at least one administrator account configured within a network to operate it as a `private network`. You can create new administrators and [rotate the validator set](/workflow/setup-private-network/#step-6-rotating-the-validator-accounts) in a single configuration update. The operator must first ensure the `global_state.toml` file contains new administrators. The validator set is updated after if an administrator is also a validator. Also, only the administrator accounts can hold and distribute token balances.

### Configuring administrator accounts

Use this configuration option in the `chainspec.toml` to add administrator accounts to the private network:

```toml
[core]
administrators = ["NEW_ACCOUNT_PUBLIC_KEY"] 
```

**Note**: Regular accounts are not allowed to manage their associated keys on a private network. 

### Generating new administrator accounts

Use the command below to generate new administrator accounts in your private network. This generates the contents of a `global_state.toml` with the entries required to create new administrator accounts at the upgrade point.

```sh
global-state-update-gen \
  generate-admins --data-dir $DATA_DIR/global_state \
  --state-hash $STATE_ROOT_HASH \
  --admin $PUBLIC_KEY_HEX, $BALANCE
```

- `NEW_PUBLIC_KEY` - Public key of the administrator in a hex format.
- `NEW_BALANCE` - Balance for the administrator’s main purse.
- `DATA_DIR` - Path to the global state directory.
- `STATE_ROOT_HASH` - State root hash, taken from the latest block before an upgrade.

### Managing accounts and smart contracts

Only administrators have permission to control accounts and manage smart contracts in a private network. An example implementation can be found in [Casper node's private chain control management](https://github.com/casper-network/casper-node/blob/c8023736786b2c2b0fd17250fcfd50502ff4151f/smart_contracts/contracts/private_chain/control-management/src/main.rs) file. This is not an existing contract. You can use the existing client contracts as an administrator to perform actions as a user. This is done by sending a deploy under a regular user's public key but signed using the administrator's secret key. 

Use this command to generate these contracts:

```sh
make build-contracts-rs
```

Only the administrator can use the related Wasm to send the deploy to the network and then use it to manage, enable, and disable contracts. This is achieved through entry points that handle enabling and disabling options for account and smart contracts:

- **To disable a contract**: Execute the `disable_contract.wasm` with `contract_hash`and `contract_package_hash` as parameters.
- **To enable a contract**: Execute the `enable_contract.wasm` with `contract_hash`and `contract_package_hash` as parameters.
- **To disable an account**: Execute `set_action_thresholds.wasm` with argument `deploy_threshold:u8='255'` and `key_management_threshold:u8='255'`.
- **To enable an account**: Execute `set_action_thresholds.wasm` with `deploy_threshold:u8='1'` set to 1 and `key_management_threshold:u8='0'`.


## Step 5. Starting the Casper Node
After preparing the administrator accounts and validator nodes, you should start and run the Casper node to see the changes. Use this command to start the node:

```sh
sudo systemctl start casper-node-launcher
```
Refer to the [Casper node setup GitHub](https://github.com/casper-network/casper-node/tree/master/resources/production#casper-node-setup) guide to know more details about configuring a new node to operate within a network.

Additionally, refer to the [casper-node-launcher](https://github.com/casper-network/casper-node-launcher) to check whether the installed node binaries match the installed configurations by comparing the version numbers.

## Step 6. Rotating the Validator Accounts
You need to go through [setting up a validator node ](/#step-1-setting-up-the-validator-nodes) guide before starting this section.

Use this command to create content in the `global_state.toml` file to rotate the validator set. Specify all the current validators, their stakes, and new accounts. 

```sh
global-state-update-gen \
  validators --data-dir $DATA_DIR/global_state \
  --state-hash $STATE_ROOT_HASH \
  –validator $PUBLIC_KEY_HEX,$STAKE,$OPTIONAL_DELEGATION_RATE
``` 

To rotate the validators set, perform a network upgrade with a `global_state.toml` with new entries generated by the `global-state-update-gen` command.

You can find more details on enabling new validators in the [joining a running network](/operators/joining/) guide. The guide explains how to join the network and provide additional security to the system. 


## Step 7. Testing the Private Network
We will describe the testing flow using an example customer and the configuration below. These options are relative to this example customer.

### Sample configuration files

Here are sample configurations that can be adapted for testing:
- A [chainspec template](https://github.com/casper-network/casper-node/blob/0b4eead8ea1adffaea98260fe2e69dfc8b71c092/resources/private/chainspec.toml.in) that is specific to the customer's private chain.
- An [accounts template](https://github.com/casper-network/casper-node/blob/0b4eead8ea1adffaea98260fe2e69dfc8b71c092/resources/private/accounts.toml.in) with one administrator in the `administrators` settings.


### Specifying IP addresses

Here is an example set of IP addresses in use:
```
http://18.224.190.213:7777
http://18.188.11.97:7777
http://18.188.206.170:7777
http://18.116.201.114:7777
```

### Setting up the node

Set up the node address, chain name, and the administrator's secret key. 

```sh
export NODE_ADDR=http://18.224.190.213:7777
export CHAIN_NAME="private-test"
```

This testing example will also use an `alice/secret_key.pem` file, a secret key generated through the [keys generation process](/dapp-dev-guide/keys/#creating-accounts-and-keys). Alice is a regular user in this testing example.

### Funding Alice's account

The following command transfers tokens to Alice's account.

```sh
casper-client \
  transfer \
  -n $NODE_ADDR \
  --chain-name $CHAIN_NAME \
  --secret-key admin/secret_key.pem \
  --session-account=$(<admin/public_key_hex) \
  --target-account=$(<alice/public_key_hex) \
  --amount=100000000000 \
  --payment-amount=3000000000 \
  --transfer-id=123
```

To check the account information, use this command:

```sh
casper-client get-account-info -n $NODE_ADDR 
  --public-key alice/public_key.pem
```

### Adding a bid as Alice
The following command attempts to add an auction bid on the network. It should return `ApiError::AuctionError(AuctionBidsDisabled) [64559]`.


```sh
casper-client \
  put-deploy \
  -n $NODE_ADDR \
  --chain-name $CHAIN_NAME \
  --secret-key alice/secret_key.pem \
  --session-path add_bid.wasm \
  --payment-amount 5000000000 \
  --session-arg "public_key:public_key='$(<alice/public_key_hex)'" \
  --session-arg "amount:u512='10000'" \
  --session-arg "delegation_rate:u8='5'"

# Error: ApiError::AuctionError(AuctionBidsDisabled) [64559]"
```

We should get a similar error for the delegate entry point.

### Disabling Alice's account

The following command disables Alice's account. In this case, executing deploys with Alice's account will not be successful.

```sh
casper-client \
  put-deploy \
  -n $NODE_ADDR \
  --chain-name $CHAIN_NAME \
  --secret-key admin/secret_key.pem \
  --session-account=alice/public_key_hex
  --session-path set_action_thresholds.wasm \
  --payment-amount=2500000000 \
  --session-arg "key_management_threshold:u8='255'" \
  --session-arg "deploy_threshold:u8='255'"
```


### Enabling Alice's account
The following command enables Alice's account. In this case, executing deploys with Alice's account will be successful.

```sh
casper-client \
  put-deploy \
  -n $NODE_ADDR \
  --chain-name $CHAIN_NAME \
  --secret-key admin/secret_key.pem \
  --session-account=alice/public_key_hex
  --session-path set_action_thresholds.wasm \
  --payment-amount=2500000000 \
  --session-arg "key_management_threshold:u8='0'" \
  --session-arg "deploy_threshold:u8='1'"
```


### Enabling a contract

The following command enables a contract using its hash.

```sh
casper-client \
  put-deploy \
  -n $NODE_ADDR \
  --chain-name $CHAIN_NAME \
  --secret-key admin/secret_key.pem \
  --session-account=$(<alice/public_key_hex) \
  --session-path enable_contract.wasm \
  --payment-amount 3000000000 \
  --session-arg "contract_package_hash:account_hash='account-hash-$CONTRACT_PACKAGE_HASH'" \
  --session-arg "contract_hash:account_hash='account-hash-$CONTRACT_HASH'"
```


### Disabling a contract

The following command disables a contract using its hash. Executing this contract using `CONTRACT_HASH` again should fail.

```sh
casper-client \
  put-deploy \
  -n $NODE_ADDR \
  --chain-name $CHAIN_NAME \
  --secret-key admin/secret_key.pem \
  --session-account=$(<alice/public_key_hex) \
  --session-path disable_contract.wasm \
  --payment-amount 3000000000 \
  --session-arg "contract_package_hash:account_hash='account-hash-$CONTRACT_PACKAGE_HASH'" \
  --session-arg "contract_hash:account_hash='account-hash-$CONTRACT_HASH'"
```
Alice needs a container access key for the contract package in her named keys.

### Verifying seigniorage allocations

[Seigniorage](https://www.investopedia.com/terms/s/seigniorage.asp) allocations should be zero at each switch block. This is the related configuration:

```toml
[core]
compute_rewards = false
```

Validator stakes should not increase on each switch block. Run this command to verify this:

```sh
casper-client get-era-info -n $NODE_ADDR -b 153
```

The total supply shouldn't increase, and the validator's stakes should remain the same.


### Rotating validators

The following command rotates the validator set. Perform a network upgrade with a `global_state.toml` with the new entries generated by the `global-state-update-gen` command.

```sh
global-state-update-gen validators \
  --data-dir $DATA_DIR \
  --state-hash $STATE_ROOT_HASH \
  --validator NEW_PUBLIC_KEY,NEW_STAKE \
  --validator NEW_PUBLIC_KEY2,NEW_STAKE2
```

### Adding new administrators
The following command produces the administrator content in the `global_state.toml` file. 

```sh
global-state-update-gen generate-admins --admin NEW_PUBLIC_KEY,NEW_BALANCE --data-dir $DATA_DIR --state-hash $STATE_ROOT_HASH
```
Remember that new administrators can be created, and the validator set can also be rotated in a single update. 

The `chainspec.toml` file should contain the following entries that include new administrators as well as existing ones for an upgrade:


```toml
[core]
administrators = ["NEW_PUBLIC_KEY"]
```

After this step, the private network would be ready for use.

