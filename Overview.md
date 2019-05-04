# DigiByte

DigiByte is the forerunner and by far the most widely used cryptocurrency. In what follows we will use the customary notation of referring to the DigiByte network/protocol as **DigiByte** (Capitalized) and to the units of currency flowing in the DigiByte network as **DigiBytes** (lowercase). Sometimes we will be using  smaller units called **[satoshis](notes-and-glossary#satoshis)**.

By design, DigiBytes have several highly desired technical attributes
* Fixed supply
* Non-counterfeitable
* Divisible
* Easily transportable

On top of that, DigiByte is decentralized. Decentralization adds an extra layer of robustness because the DigiByte network has no single point of failure.

Any and all DigiByte transactions are publicly available on the a ledger a.k.a “The Blockchain”, and are  [immutable](notes-and-glossary#immutability).

# DigiAssets

While originally designed to be a currency, DigiByte supports a limited scripting language that can be used to store metadata on the blockchain. DigiAssets is a concept that allows attaching metadata to DigiByte transactions and leveraging the DigiByte infrastructure for issuing and trading immutable digital assets that can represent real world value. The value of such digital assets is tied to a real-world promise by the asset issuers that they are willing to redeem those digital tokens for something of value in the real world. 

Digital asset on top of the DigiByte's Blockchain can be used to issue Financial assets (securities like shares, commodities like Gold or new currencies), Prove of ownership (A digital key to a house or a car, a concert ticket), store information (Documents, Certificates, Contracts) or creating smart contracts (CFDs, Loans, Bets, Derivatives).

The advantage given by using the blockchain as the backbone for such asset manipulation is that one can rely on the blockchain’s transparency, immutability, ease of transfer and non-counterfeitability to transfer and trade such digital tokens with unprecedented security and ease.

# Desired attributes of a DigiAssets protocol
* Embed the maximal amount of data with minimal amount of load on the blockchain
* Be decentralized
* Allow for sending arbitrary amounts of metadata with every DigiAsset transaction
* Allow for both [locked assets](Benefits#locked-assets) (can be reissued) and [unlocked assets](Benefits#unlocked-assets) (cannot be reissued)

# Implementations
Over the past few years we have seen several DigiAssets implementations (which are part of the larger [Bitconi 2.0](http://techcrunch.com/2014/04/19/DigiByte-2-0-unleash-the-sidechains/) movement.
* [colu](http://www.colu.co)
* [openassets](https://www.coinprism.com/)
* [chromawallet](http://chromawallet.com/)
* [coinspark](http://coinspark.org/)

# The DigiAssets Protocol
This document describes the DigiAssets protocol specification for issuing and transacting digital asset on top of the DigiByte Blockchain.

This Document consists of two main parts
## 1. The DigiAsset scheme  
The DigiAsset scheme describes the encoding used to encode/decode the DigiAssets data that is stored in a DigiByte transaction after the OP_RETURN operator (and sometimes in one additional multisignature output). This DigiAsset scheme is optimized for robust functionality by inserting the most data within the current size constraints of the OP_RETURN operator (80 Bytes).

## 2. Asset Metadata and the Rule engine  
This document introduces the first specification for handling metadata for digital assets.
Because of the 80 Bytes limit of the OP_RETURN field, any metadata that is attached to a transaction is hashed and stored in torrents to allow decentralized access to the information.
By hashing the metadata for every transaction, DigiAssets actually expands dramatically the basic DigiByte functionality by allowing to insert rules that exist outside the blockchain and can be publicly accessed via torrents and verified against a specific transaction thus creating a rule engine with basic smart contract functionality.