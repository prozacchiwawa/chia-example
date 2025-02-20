# Test out the Piggybank Example in 15 minutes

In this example we will:
 - Connect to the Chia testnet
 - Create a wallet and add some mojos to it
 - Create a copy of the example piggybank contract such that our wallet is the cash out destination where funds will be deposited when the savings goal is reached
 - Deploy the contract 
 - Send some of our test mojos to the piggybank
 - Verify the piggybank's value has increased
 - Verify that the piggybank cashes out to our wallet when the savings goal is reached

## Connect to testnet
In order to deploy our coin to the testnet, we want to run a full node. This will also allow us to test out all the other chia CLI commands to directly inspect the blockchain state. 

### Create and sync a wallet
The Clovyr Code environment comes with the testnet10 blockchain database pre-downloaded. The first step is to start the node, which will then connect to peers and download the latest activity since the last database image. Syncing to the current block height usually takes less than five minutes. 

1. `chia start node` - start the node
   - `chia show -c` - view peers (more peers will be added over time)
   - `chia show -s` - view sync status 
2. `chia keys generate` - generate unique keys private to the user
3. `chia start wallet` - begin the wallet fast sync

### Get test mojos
1. `chia wallet show` - view wallet fingerprint and sync status
2. `chia wallet get_address -f [fingerprint]` - get wallet address from fingerprint
3. https://testnet10-faucet.chia.net/ - open the chia testnet web faucet
   - TODO: REPLACE WITH COMMANDS AFTER API INTEGRATION
4. enter the wallet address and click [Submit]
5. `chia wallet show` - verify that the txch was received (this takes about a minute)

## Update piggybank example with our wallet address
 - `cp chia-piggybank piggybank-test` - copy the chia-piggybank folder for this test
 - open file `piggybank-test/piggybank.clsp`
 - `cdv decode [wallet address]` - decode bech32m address to a puzzle hash
 - update `CASH_OUT_PUZZLE_HASH` constant with the puzzle hash of your wallet. 
    - *Prepend the address with "0x!"* (just like in the example contract)
 - save the file

## Deploy empty piggybank
This script compiles the piggybank.clsp file to clvm, gets its puzzle hash, and forms a coin with zero mojos and this puzzle hash. 

- `python3 -i ./piggybank_drivers.py` - load the piggybank python driver in interactive mode
- `piggybank = deploy_smart_coin("piggybank.clsp", 0)` - deploy the piggybank contract with initial balance of 0

- Note `your_puzzle_hash`

## Move mojos into piggybank
We create a contribution coin, which spends funds from our wallet into a contribution coin. Contribution coins are coins earmarked for a specific purpose. In the future, we will add code to contribution.clsp that restricts the coin so that it will only spend itself as a deposit into the piggybank. 

 - `contribution_100 = deploy_smart_coin("contribution.clsp", 100)`
 - `contribution_200 = deploy_smart_coin(("contribution.clsp", 450)`
 - `deposit(piggybank, contribution_100)`

## Verify savings dump to new coin
 - `CTRL + D` to exit the python interpreter (will unset the values of `piggybank`, `contribution_100`, & `contribution_200`)
 - `cdv rpc coinrecords --by puzhash [your_puzzle_hash] -ou -s 460000 -nd`
