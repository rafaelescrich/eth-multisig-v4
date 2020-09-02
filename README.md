# Ethereum MultiSig Wallet Contract

## About

Multi-sig contract suitable for use in wallets. 

Some of the features of the contract (WalletSimple.sol)

1. Functions as a 2-of-3 multisig wallet for sending transactions.
2. Support for synchronous (single transaction) approvals containing multiple signatures through the use of ecrecover.
3. Can deploy Forwarder contracts so that a single wallet can have multiple receive addresses. 
4. Forwarder address contracts have the ability to flush funds that were sent to the address before the contract was created.
5. ERC20 tokens can be flushed from the forwarder wallet to the main wallet with a single signature from any signer.
6. ERC20 tokens and ether can be sent out from the main wallet through a multisig process.
7. ‘Safe Mode’ can be set on a wallet contract that prevents ETH and ERC20 tokens from being sent anywhere other than to wallet signers.
8. Transactions can be sent in batches through the [Batcher](contracts/Batcher.sol) contract to save on transfer fees.

### Deployment
The Wallet contract and forwarder contracts can each be deployed independently of each other, using the provided ForwarderFactory and WalletFactory.
These factories employ two features to minimize the cost associated with deploying a new contract:
- [Minimal proxy](https://eips.ethereum.org/EIPS/eip-1167) - Each deployed contract is simply a tiny contract which proxies calls to use the logic of a single implementation contract.
- [CREATE2](https://eips.ethereum.org/EIPS/eip-1014) - Contracts are deployed with the CREATE2 opcode to allow users to distribute contract addresses and only deploy them upon first use. 

**Wallets**
To deploy wallets, follow these steps:
1. Deploy a wallet contract ([contracts/WalletSimple.sol](contracts/WalletSimple.sol)) with any address. Take note of the wallet's address.
2. Deploy a WalletFactory contract ([contracts/WalletFactory.sol](contracts/WalletFactory.sol)) with any address. Use the address of the contract deployed in step 1 as the `_implementationAddress` parameter.
3. Call the `createWallet` function on the factory deployed in step 2. Provide the list of signers on the wallet, and some "salt" which will be used to determine the wallet's address via [CREATE2](https://eips.ethereum.org/EIPS/eip-1014).
4. Check for the `WalletCreated` event from the above transaction. This will include your newly generated wallet address

**Forwarders**
To deploy forwarders, follow these steps:
1. Deploy a forwarder contract ([contracts/Forwarder.sol](contracts/Forwarder.sol)) with any address. Take note of the contract's address.
2. Deploy a ForwarderFactory contract ([contracts/ForwarderFactory.sol](contracts/ForwarderFactory.sol)) with any address. Use the address of the contract deployed in step 1 as the `_implementationAddress` parameter.
3. Call the `createForwarder` function on the factory deployed in step 2. Provide the parent address, and some "salt" which will be used to determine the forwarder's address via [CREATE2](https://eips.ethereum.org/EIPS/eip-1014).
4. Check for the `ForwarderCreated` event from the above transaction. This will include your newly generated forwarder address


## Contracts
Brief descriptions of the various contracts provided in this repository.

[**WalletSimple**](contracts/WalletSimple.sol)

The multi-sig wallet contract. Initializes with three signers and requires authorization from any two of them to send a transaction.

[**Forwarder**](contracts/Forwarder.sol)

Forwarder function. Initializes with a parent to which it will forward any ETH that it receives. Also has a function to forward ERC20 tokens.

[**WalletFactory**](contracts/WalletFactory.sol)

Factory to create wallets. Deploys a small proxy which utilizes the implementation of a single wallet contract.

[**ForwarderFactory**](contracts/ForwarderFactory.sol)

Factory to create forwarder. Deploys a small proxy which utilizes the implementation of a single forwarder contract.

[**Batcher**](contracts/Batcher.sol)

Transfer batcher. Takes a list of recipients and amounts, and distributes ETH to them in a single transaction. 

## Installation

NodeJS 8.14.0 is recommended.

```shell
npm install
```

This installs truffle and an Ethereum test RPC client.

## Wallet Solidity Contract

Find it at [contracts/WalletSimple.sol](contracts/WalletSimple.sol)

## Running tests

A test suite is included through the use of the truffle framework, providing coverage for methods in the wallet.

The truffle framework will depend on the Web3 interface to a local Web3 Ethereum JSON-RPC. If you've followed the above steps, run the following to start testrpc. 

```shell
npm run truffle-testrpc
```

You should verify that you are not already running geth, as this will cause the tests to run against that interface. 

In a **separate terminal window**, run the following command to initiate the test suite, which will run against the RPC:

```shell
npm run truffle-test
```

## Notes
- wallet creation salt should include [a hash of] the signers associated with the wallet. 
- forwarder creation salt should include [a hash of] the parentAddress.
