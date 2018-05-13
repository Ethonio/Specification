# Specification
Ethon protocol specification


## Consensus
### Tempo consensus
The hash of a node address must end with an amount of zero's to prevent sybil attacks. Multiple trusted parties an decide to use the same node address to reduce costs.

## Cross shard communication
### Example
Alice sends 1 ether to an exchange contract. The exchange contract calls an ERC20 contract to transfer a token to alice's account.

Alice's account is on shard 1.
The exchange contract resides on shard 2.
The ERC20 contract resides on shard3.

1. Alice sends an execute atom to shard 1
2. Shard 1 communicates this with shard 2 and blocks
  1. Shard 2 starts executing exchange contract
  2. Exchange contract runs CALLVALUE and receives a 1
  3. Exchange contract runs CALL to ERC20 contract on shard3, blocks      
    1. ERC20 contract checks CALLER, receives exchange contract address
    2. ERC20 runs CALLDATALOAD to read amount and destination address
    3. ERC20 writes the transfer to it local datastore
    4. ERC20 returns
  4. Exchange contract unblocks
3. Shard 1 updates balance, unblocks


## Ethereum vm
[Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) 

REVERT operation is difficult to implement

### Opcodes
Different operations can be found starting from [page 28](https://ethereum.github.io/yellowpaper/paper.pdf#page=28) of the yellow paper.

Special cases:
30s - Environment Information
* 0x31 BALANCE - Get an account's Balance
* ...

40s - Block information

f0s - System operations
* 0xf0 CREATE - Create a new account with associated code
* ...

### Sharding
Each ethereum address has a designated shard. Any interaction between contracts is thus wrapped in the cross-chain protocol. While a shard waits for a response, the VM execution is blocked.

Perhaps the shardspace should be equal to the address space?

## Relay
Specifications for the duplex relay between the Ethereum mainnet and Ethon.
