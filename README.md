# Ethereum-tutorial

Ethereum is a secure decentralized platform facilitating arbitrarily complex computations. For an overview, read the [white paper](https://github.com/ethereum/wiki/wiki/White-Paper). For detail, read the [yellow paper](http://gavwood.com/paper.pdf).

This tutorial aims to reproduce our own approach towards understanding the command-line tools and the Solidity language for smart contract. Please note this is a work in progress.

There are many different tools providing access to the Ethereum network. It is instructive to stick with the command-line tools to start out. Specifically, we will use `geth`, the Go implementation of the command-line interface, to run nodes on the network.


# Installation

### Ubuntu ###
To install `geth` from PPA, run:

```bash
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

### Mac ###
If you don't have `brew`, install it:
		
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
To install `geth`, run:

```bash
brew tap ethereum/ethereum
brew install ethereum
```


# Starting a private node
To keeps things simple for now, we will start by running a single node on a private network. This will allow us to inspect the basic data structures, objects, and procedures without the delay and complications of hopping onto an existing large-scale blockchain. To do this, we will use a custom genesis block. This is done by specifying the following key block details as a JSON file.

* nonce: 64-bit hash used with mixhash for proof-of-work
* timestamp: Unix-time value
* parentHash: Keccak 256-bit hash of parent block header
* gasLimit: Maximum amount of gas that can be expended in a single block
* difficulty: Determines the difficulty of mining this block
* mixhash: 256-bit hash used with nonce for proof-of-work
* coinbase: The address of the account in which mining rewards are deposited
* alloc: A specification of initial Ether allocations among accounts on the network
* extraData: Additional relevant block data, up to a maximum of 32 bytes

Save the following into a file called 'genesis.json'.

```json
{
	"nonce": "0x0000000000000123",
	"timestamp": "0x0",
	"parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	"extraData": "0x0",
	"gasLimit": "0x8000000",
	"difficulty": "0x400",
	"mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
	"coinbase": "0x3333333333333333333333333333333333333333",
	"alloc": {}
}	
```

To start a node, run

	geth --genesis genesis.json

This will use the above json file to write block 0 of the blockchain, the genesis block. You will see
some networking logs indicating the server starting. Open another terminal and run

	geth attach

This will launch the Javascript Runtime Environment included in `geth`. Here, you have access to a
variety of management APIs, and you can write ordinary JavaScript code. It is instructive to keep
an eye on both terminal windows simultaneously, as the first provides immediate insights for debugging
and understanding.

Start by inspecting the state of the network.

```javascript
> web3.eth.accounts
[]
> web3.eth.coinbase
null
> web3.eth.blockNumber
0
> web3.eth.getBlock(0)
{
  difficulty: 1024,
  extraData: "0x00",
  gasLimit: 134217728,
  gasUsed: 0,
  hash: "0x8b1f2271ac3d51f7ca371b8e633e8f1625b64fb4bad3a158cc3da8157dfdaa14",
  logsBloom: "0x00000000000000000000000000...",
  miner: "0x3333333333333333333333333333333333333333",
  nonce: "0x0000000000000123",
  number: 0,
  parentHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
  receiptRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  sha3Uncles: "0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347",
  size: 507,
  stateRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  timestamp: 0,
  totalDifficulty: 1024,
  transactions: [],
  transactionsRoot: "0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  uncles: []
}
> web3.eth.getBlock(1)
null
```

So, we have no accounts, our coinbase (default recipient of mined Ether) is null, we are on block number 0, and we can see within its information the data from our JSON file.

### Accounts ###
Everything in Ethereum starts with an account. The first type of account is called an *externally owned account*, and is created like so:

```javascript
> web3.personal.newAccount("strong password that no one will guess")
"0x7a47376c00308be583e1ea5e7a610d58dce68744"
> web3.personal.newAccount("haha")
"0xd451d8e4cc7392c9a21ac8c0e23c6beac1343691"
> web3.eth.accounts
["0x7a47376c00308be583e1ea5e7a610d58dce68744", "0xd451d8e4cc7392c9a21ac8c0e23c6beac1343691"]
> web3.eth.getBalance(web3.eth.accounts[0])
0
```

We will see the other type of account, a *contract account*, when we get to writing smart contracts in Solidity.

### 🔨 Mining ###
To do anything, we need Ether. 
		
```javascript
> miner.start()
	true
```

In the other window, you will see something like this (the timestamps and program name have been ommitted to save space):

```bash	
Starting mining operation (CPU=8 TOT=9)
Automatic pregeneration of ethash DAG ON (ethash dir: /home/momo/.ethash)
commit new work on block 1 with 0 txs & 0 uncles. Took 110.399µs
checking DAG (ethash dir: /home/momo/.ethash)
Generating DAG for epoch 0 (size 1073739904) (0000000000000000000000000000000000000000000000000000000000000000)
Generating DAG: 0%
Generating DAG: 1%
Generating DAG: 2%
Generating DAG: 3%
Generating DAG: 4%
Generating DAG: 5%
Generating DAG: 6%
Generating DAG: 7%
Generating DAG: 8%
Generating DAG: 9%
Generating DAG: 10%
```

Once the [DAG](https://github.com/ethereum/wiki/blob/master/Dagger-Hashimoto.md) is generated, mining will start:

```bash
Generating DAG: 99%
Generating DAG: 100%
Done generating DAG for epoch 0, it took 4m18.16510184s
Starting mining operation (CPU=8 TOT=9)
commit new work on block 1 with 0 txs & 0 uncles. Took 129.44µs
🔨  Mined block (#1 / 44d47c8c). Wait 5 blocks for confirmation
commit new work on block 2 with 0 txs & 0 uncles. Took 150.156µs
🔨  Mined block (#1 / 348b5a21). Wait 5 blocks for confirmation
commit new work on block 2 with 0 txs & 0 uncles. Took 123.073µs
commit new work on block 2 with 0 txs & 0 uncles. Took 977.309µs
🔨  Mined stale block (#1 / 18c10b67). 
commit new work on block 2 with 0 txs & 0 uncles. Took 191.801µs
commit new work on block 2 with 0 txs & 0 uncles. Took 142.205971ms
🔨  Mined block (#1 / d11b9cb6). Wait 5 blocks for confirmation
commit new work on block 2 with 0 txs & 0 uncles. Took 28.898533ms
commit new work on block 2 with 0 txs & 0 uncles. Took 122.455µs
🔨  Mined block (#2 / cfb278cb). Wait 5 blocks for confirmation
commit new work on block 3 with 0 txs & 0 uncles. Took 205.311µs
🔨  Mined stale block (#2 / ac569b95). 
commit new work on block 3 with 0 txs & 0 uncles. Took 146.449742ms
commit new work on block 3 with 0 txs & 0 uncles. Took 156.201µs
🔨  Mined block (#3 / 06152682). Wait 5 blocks for confirmation
```

By default, all mining rewards are deposited into the account at `web3.eth.accounts[0]`. In the console, you can verify the increasing balance:


```javascript
> web3.getBalance(web3.eth.accounts[0]) //units are in wei. 1 ether = 10^18 wei
45000000000000000000
> web3.getBalance(web3.eth.accounts[1]) // still 0
0
> web3.fromWei(web3.getBalance(web3.eth.accounts[0])) // convert from wei to ether
45
```

To send some Ether, run:

```javascript
> var sender = web3.eth.accounts[0]; var receiver = web3.eth.accounts[1]
undefined
> web3.personal.unlockAccount(sender, "strong password that no one will guess")
true
> web3.eth.sendTransaction({from: sender, to: receiver, value: 20})
"0xcd2b5724cff9afcf5236a1f49692fbc9082ffac5da6cbe176626c2f8443f07a8"  //transaction hash
```

In the other window, you will see the transaction hash and destination address, followed by a log of a block mined with `1 txs`, a single transaction.

```bash
Tx(0x2d4e0ec822f65549116da31cb70d6d04b39d558e031b4927927a75cec87030ff) to: 0xd451d8e4cc7392c9a21ac8c0e23c6beac1343691
 commit new work on block 18 with 1 txs & 0 uncles. Took 793.931µs   // our transaction will be mined in block 18
  🔨  Mined block (#18 / 57a17fa2). Wait 5 blocks for confirmation     // our transaction has been mined
 commit new work on block 19 with 0 txs & 0 uncles. Took 232.13µs
 commit new work on block 19 with 0 txs & 0 uncles. Took 260.54µs
 🔨  Mined block (#19 / 6eae0bae). Wait 5 blocks for confirmation
 commit new work on block 20 with 0 txs & 0 uncles. Took 195.227µs
 commit new work on block 20 with 0 txs & 0 uncles. Took 183.743µs
```

Confirm the payment has been received:

```javascript
> web3.eth.getBalance(receiver)
20
```

### Contracts ###

### Node.js ###
