# Data storage on the blockchain

There are several ways to store data on top of the DigiByte blockchain.
#### Value
Encode data in the number of satoshis being sent to an address.

#### Fake Addresses
Encode data in the Address itself. Because the Address encodes data of your choice it cannot have been the result of a derivation from a private key (with extremely high probability) and thus any coins sent to such addresses are lost (or "burnt").

#### Vanity Addresses
Brute force through keys until you get an address that encodes your data, extremely resource intensive and impractical for anything bigger than a couple of bytes.

#### 1 of N Multisig Address
These are more complex DigiByte addresses that require one key out of N to redeem. We can use only one key as a real key (like with a standard address) and encode 32 bytes of data in the remaining `N-1` keys.

#### OP_RETURN
OP_RETURN is a command in the DigiByte scripting language that was specifically added to allow the inclusion of metadata on the blockchain. Currently 80 bytes of information can be added to a transaction using OP_RETURN.

#### Input Sequence
This is an unused 32 bit integer .

#### Coinbase
Miners can include up to 100 bytes worth of data in a coinbase transaction.
 
## Comparison of Data Storage Methods
| Method                      | Pros                   | Cons                                          | 
| ----------------------------|------------------------|---------------------------------------------- |
| **Value**                   | Very Simple            | **Expensive**, temporary burden on the network|
| **Vanity** Address          | Simple | Resource intensive, **impractical** for more than a few bytes | 
| **Fake** Address            | Simple, 20 bytes          | Lost coins, permanent burden on the network| 
| 1 of N **Multisig** Address | `32x(N-1)` bytes    | More **complex**, temporary burden on the network|
| **OP_RETURN**               | 80 bytes, no burden on the network |                                   |
| **Sequence**                |                                    | Only 4 bytes                      |
| **Coinbase**                | Up to 100 bytes                    | Can only be used by **Miners**    |

