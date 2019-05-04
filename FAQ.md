In this section we provide answers to some frequently asked questions about the DigiAssets protocol.

# DigiAsset Micropayment Channels?
A [Micropayment channel](https://bitcoin.org/en/developer-guide#micropayment-channel) is a mechanism that allows two parties to be engaged in a contract that gets updated every so often without continuously transmitting (relatively) costly transactions to the DigiByte network. Micropayment channels are established using a deposit to a [2 of 2 multisignature](DigiAsset%20Scheme#multisignature-addresses-multisig) address (and making use of the the DigiByte [NLockTime](https://en.DigiByte.it/wiki/NLockTime) parameter). Basically there are:
* A time delayed **refund** transaction: returning all the funds back to the client after a set time (say 1 day) if the service provider fails to respond.
* A continuous exchange of signed standard (i.e. non-delayed) transactions which send incrementally more to the service provider at the expense of the client. 

The primary use case is for transacting small amount of money for continuous services like video or music streaming or an internet connection.
 
Because DigiAssets run on top of the DigiByte network, DigiAsset micropayment channels can be implemented in exactly the same way if the transactions (i.e. the delayed refund transaction and the continuously updated payment transactions) are DigiAsset transactions transferring units of an asset instead of DigiBytes.

Note also that using our [rule engine](Rules) and particularly the [expiration](Rules#expiration) feature we can also implement micropayments without relying on the [standard technology](https://DigiBytej.github.io/working-with-micropayments) described above.

# Pruning OP_RETURN?

A possible concern about relying on OP_RETURN outputs is [blockchain pruning](https://en.DigiByte.it/wiki/Scalability#Storage). Since the data after an OP_RETURN command is basically ignored by the DigiByte scripting language, OP_RETURN outputs may safely be dropped (pruned) in future attempts to save blockcahin space. We believe that this is not a problem in practice. In fact, embedding metadata on the DigiByte blockchain is only finding more [use cases](http://www.coindesk.com/DigiByte-core-dev-update-5-transaction-fees-embedded-data/) and getting [more traction](http://www.slideshare.net/coinspark/DigiByte-2-and-opreturns-the-blockchain-as-tcpip/30) as time goes by. We thus believe that practical future attempts to reduce blockchain size will not remove OP_RETURN outputs. Furthermore, we plan on hosting our own custom DigiAsset coin node(s) in the near future, those nodes will definitely not prune OP_RETURN outputs.

# Securing Issuance Private Keys?
[Unlocked assets](Benefits#unlocked-assets) create a special risk fundamental to the concept of DigiAssets. 
If someone steals the private keys of a standard DigiByte address, all you can loose is the coins in that address. However, if someone steals the private keys that control an issuing address it is much worse. For example, if I have issued 100 units of an asset and commit to pay 1 DigiByte for each unit, someone who steals my key can issue 1 million additional units, sell them, and leave me in debt of a million DigiBytes.

The DigiAssets protocol gives the option of issuing [Locked assets](Benefits#locked-assets) which circumvent this issue by sacrificing some flexibility. Issuers of unlocked assets should be aware of that risk and mitigate it with real world measures such as insurance or an explicit clause in the contract that limits their liability in such cases.  

# DigiAsset [Satoshis](http://DigiByte.stackexchange.com/a/117)?
The DigiAssets idea has evolved over time. Initially the idea was to "color satoshis", meaning that actual satoshis were labeled and represented value, hence "DigiAsset". As the protocol evolved it was realized that the essence of the DigiAssets protocol was not attaching values to satoshis but rather encoding a logical data layer that supports the issuance and transfer of digital assets on top of the DigiByte blockchain. The DigiByte blockchain provides the underlying security and transferability layer upon which digital assets are created and moved around. 

In the modern implementation of the DigiAssets protocol actual satoshis indeed move around, but only in order to pay for the use of the DigiByte blockchain. All the data necessary for asset manipulation is [encoded](DigiAsset Scheme) and there is no longer any 1:1 correspondence between assets and satoshis.

However, while not DigiAsset satoshis, one can still say that the the DigiAssets protocol "colors UTXOs". 

## DigiAsset UTXOS
The DigiByte protocol keeps track of DigiBytes in an unorthodox way. Unlike traditional bank accounts where the basic entity is an **account** and balance moves from account to account. In DigiByte, the basic entity is a [UTXO](https://bitcoin.org/en/glossary/unspent-transaction-output) which is short for Unspent Transaction Output. 

A UTXO can be thought of as an IOU note, or a check, authorizing the right person (e.g. holder of the private key to an address) to release DigiBytes that can be used in a new transaction. Just like IOUs, UTXOs cannot be partially used, you can either use it all or not (deposit the check or not). From the perspective of the DigiByte protocol, the balance in an address is a derived concept, equivalent to the sum of DigiBytes credited in UTXOs to that address. In the bank analogy, think of a situation where there are no accounts, just many checks. Your "balance" is then the sum of values in valid checks that where written for you as a recipient. In order to use your money, you can no longer withdraw an arbitrary amount from your account, instead, you pick a subset of checks credited to you whose total value exceeds that sum you want to "withdraw" and cache them. 

A UTXO is uniquely determined by two pieces of data: 
* The transaction id 
* The index of the output in that transaction

The DigiAssets protocol uses the same perspective. DigiAssets UTXOs authorize the right person to transfer an Asset (or Assets) in a DigiAsset transaction. The DigiAssets protocol uses the data in the OP_RETURN output to encode asset values in DigiByte UTXOs. DigiAsset UTXOs can be used as inputs in DigiAsset transaction, facilitating asset issuance and transfer. Similarly to the way that a DigiByte UTXO can be traced back all the way to a [coinbase transaction](https://bitcoin.org/en/glossary/coinbase), a DigiAsset UTXO can be traced all the way back to an [issuance transaction](DigiAsset-Scheme#issuance-and-transfer-transactions).

# How many assets can be in one Address?
The answer to that depends on the asset type.
If the asset is [Locked](https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Benefits#locked-assets) we can have an unlimited number of assets in one address. On the other hand, there can be **at most one** [Unlocked](https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Benefits#unlocked-assets) asset in an address. The reason is that [Unlocked AssetIDs](https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Asset%20ID#unlocked-asset-ids) are tied to the address and thus issuing again is considered to be a **reissuing** of the same asset.