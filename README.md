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

1. Alice sends an execute atom to shard 1 with some ether
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

### Two-phase locking
Ethereum has a max stack depth of 1024 CALLs. Interesting talk on 

* Keep track of all shards that have been entered, these become part of the "serial" environment (if a cyclic call is made, it can thus be executed)
* If a locked shard is encountered, revert state, unlock (to avoid deadlocks) and try again (after a timeout).
https://ethresear.ch/t/cross-shard-contract-yanking/1450 

See: https://ethresear.ch/t/cross-shard-locking-scheme-1/1269
A shard "pulls" a contract's state, operates on it and then "pushes" the result.
=> Here all referenced conracts would be "yanked" to the calling shard.

State roots are saved before the execution/locking, on revert, the state roots are simply reset.


## Ethereum vm
[Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf) 
Off chain computation: TrueBit (works with challanging)

[ewasm](https://github.com/ewasm) for vm?
[primea](https://github.com/primea) for communication?

### Transactions
Two types:
* [__Contract creation__](https://ethereum.github.io/yellowpaper/paper.pdf#page=9) 
  * init: unlimited size byte array containing EVM code for initialisation (EVM-code format), executed on account creation, then discarded
 Contract creation that results in an OOG (out of gas) exception has no impact on state. If it succeeds, a final code creation cost is paid, proportional to the code size. Another OOG exception can be thrown here.

Note that while the initialisation code is executing, the newly created address exists but with no intrinsic body code

* [__Message call__](https://ethereum.github.io/yellowpaper/paper.pdf#page=10)
  * data: unlimited size byte array
if the execution halts in an exceptional fashion, then no gas is refunded to the caller and the state is reverted to the point immediately prior to balance transfer.
Common fields:
* Nonce
* GasPrice
* GasLimit
* to: recepient address, / ∅ for contract creation
* value: amount of gwei to transfer to recepient
* v, r, s: signature related values (ECDSA of the SECP-256k1 curve of hash of all values except these 3)

Initial tests of intrinsic validity:
* The transaction is well-formed RLP, with no additional trailing bytes
* The transaction signature is valid
* The transaction nonce is valid (equivalent to the sender account’s current nonce)
* the gas limit is no smaller than the intrinsic gas used by the transaction
* the sender account balance contains at least the cost required in up-front payment.
=> can all be checked with local data

The  execution of a valid transaction begins with an irrevocable change made to the state: the nonce of the account of the sender is incremented by one and the balance is reduced by part of the up-front cost

Perhaps a transaction should be routed to a node that has all relevant shards? This way another pre-evaluation can be done to ensure that the transaction can be executed, without going through logging.

Code execution can effect several events that are not internal to the execution state: the account’s storage can be altered, further accounts can be created and further message calls can be made.  

### Opcodes
REVERT operation is difficult to implement
Block timestamp may also be used by apps

Different operations can be found starting from [page 28](https://ethereum.github.io/yellowpaper/paper.pdf#page=28) of the yellow paper.

Special cases:
30s - Environment Information
* 0x31 BALANCE - Get an account's Balance
* ...

40s - Block information
Thought: Emulate block generation in seperate tempo space.

f0s - System operations
* 0xf0 CREATE - Create a new account with associated code
* ...

### Sharding
Each ethereum address has a designated shard. Any interaction between contracts is thus wrapped in the cross-chain protocol. While a shard waits for a response, the VM execution is blocked.

Perhaps the shardspace should be equal to the address space?

## Relay
Specifications for the duplex relay between the Ethereum mainnet and Ethon.
