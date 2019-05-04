In this section we list some of the benefits of the new implementation of the DigiAssets protocol proposed by Colu.

* [Better metadata handling](#better-metadata-handling)
* [Coherent issuance policy](#coherent-issuance-policy)
* [Better Performance & efficiency](#better-performance--efficiency)
* [Smart Contracts capabilities](#smart-contracts-capabilities)
* [Issuer Verification capabilities](#issuer-verification-capabilities)

# Better metadata handling
This protocol specifies for the first time, a clear method of creating and storing the metadata for DigiAssets transaction:
## Using Torrents
Until now, smart assets that were built on the DigiByte Blockchain, used other Blockchains to hold the metadata or stored it in a centralized DB. DigiAssets new rule engine will automatically store the metadata in torrents, where they can be freely accessed and verified.
There are important advantages to this approach:
* **Decentralization**
 * **Robustness**: Even if our servers go down, the data is not lost 
 * **Ownership**: No one *owns* the data
* **Provable immutability**: A SHA-256 hash of the metadata is (*optionally*) [stored on the blockchain](DigiAsset-Scheme#data-storage). This allows our code to verify that the data loaded using torrents is indeed the correct data.

## Metadata on every DigiAsset transaction
The new DigiAssets protocol implementation allows for adding asset metadata on every transaction, not only during issuance,  which is very useful when not all the relevant data is present at the moment of issuance.

For example, a movie theater has 500 seats. The theater can issue 500 units of a *mission-impossible-10-premiere* asset representing a ticket to the premiere of the new blockbuster in town. During the buy process of a ticket on the theater's website a client selects a seating position. Each single unit of the *mission-impossible-10-premiere* asset sent to a client's DigiAsset coin wallet already contains the metadata of the selected seat.
Another example would be adding temporary data for stocks when there some kind of an event, like restrictions, stock splits etc. 

# Coherent issuance policy
The new DigiAssets protocol implementation enforces a coherent issuance policy by supporting two types of Assets, locked and unlocked:

### Locked Assets

Locked assets have a fixed amount set at the moment of issuance. Even the asset issuer [cannot issue more units](Asset%20ID#locked) of his own locked asset.

#### Example: non-diluteable shares
For example, owning a fixed percentage of a company can be represented with locked assets. For example, a Venture Capital firm is interested in investing 2.5 million dollars in a startup, provided that it receives 25% control, even in face of future dilution of shares. The company can issue a locked *startup-x-to-the-moon* asset with an amount of, say, 10 million units and send 25% of the issued units to the VC firm upon receiving their investment. The VC can be completely confident that no dilution of it's shares is possible.

### Unlocked Assets
Issuers of unlocked assets can [keep issuing](Asset%20ID#unlocked) more units of their asset.

#### Examples
For example, let's return to the company from our [previous example](Benefits#example-non-diluteable-shares). The company can issue 7,500,000 units of a new *unlocked* asset, call it *startup-x-round-B*. This time each unit is redeemable for its proportion in the total amount of the new asset out of the existing 7,500,000 locked *startup-x-to-the-moon*. Initially each unit of the new asset is redeemable for exactly one unit of the old asset, but in case of a new round of funding the new asset can be diluted by issuing more units.

As another example, a coffee shop brand issues each month 1000 units of a free-cup-of-coffee asset, each redeemable for a free cup of coffee. Each month, a 1000 new units of the same asset are issued. 

# Better Performance & efficiency

## Processing many assets in one transaction
The new DigiAssets [protocol implementation](DigiAsset-Scheme#asset-manipulating-transactions) uses a novel encoding for [asset transfer instructions](Transfer Instructions). This encoding allows us to process many assets together in a [single transaction](DigiAsset-Scheme#asset-processing-capacity-per-transaction) (in some cases up to 18 colors, or more).

## Low Prices & minimal bloat
The new DigiAssets protocol implementation achieves a high level of data compression using the way [asset transfer instructions](Transfer Instructions) and [transfer and issuance amounts](Number Encoding) are encoded. This leads to lowering the price in DigiBytes needed to support each asset manipulating transaction as well as reducing blockchain bloat.

## Support for zero confirmation transactions
DigiByte transaction are confirmed every 10 minutes (on average). Waiting for confirmations gives a poor user experience, especially considering current standards of responsiveness expected from web and mobile applications.

The architecture of the new DigiAssets protocol implementation allows us to support asset issuance and transfer in **zero confirmations** (and in fact even within the [same transaction](DigiAsset-Scheme#issuance-transaction-encoding)), because the [Asset ID](Asset ID) only references the first UTXO in the transaction and makes no reference to a block.

While it is true that each confirmation of a DigiByte transaction increases our confidence in it, for many real world cases involving small amounts of money (a cup of coffee, a movie ticket, etc..) most users and merchants accept zero confirmation transactions. The fact that the new DigiAssets protocol implementations lends support to [manipulating assets in zero confirmation](Asset%20ID#asset-ids-and-DigiByte-confirmations) is a big boon for user experience. 

As with standard DigiByte transactions, certain users may decide to consider an asset as *valid for their business purpose* only if it has at least some number of confirmations. The DigiAssets software itself does not enforce that restriction.

## Support for thin wallets
In order to allow DigiAssets wallets to run on mobile devices, the protocol must be able to verify DigiAsset transactions without the need to run a full DigiByte node. 

Thin DigiByte wallets, a.k.a [SPV clients](https://en.DigiByte.it/wiki/Scalability#Simplified_payment_verification) are nodes in the DigiByte network that do not keep all the blockchain data but instead keep only the block headers and verify that those connect correctly and that the difficulty is high enough so that the chain can be reasonably trusted.
Full data (and Merkle branch linking to blocks) are only requested for transactions relevant to the addresses in the wallet, thus using the Merkle tree structure to prove inclusion in a block without needing the full contents of the block.
In particular, the thin client does not *verify* transaction validity but instead *deduces* that validity from the inclusion in valid blocks of the longest chain.

This general type of SPV clients **cannot**, in principle, support the DigiAssets protocol. The reason is that DigiByte miners are *color blind* and thus treat color transactions as standard DigiByte transactions and ignore the entire asset meta structure that can only be parsed and understood by nodes running the DigiAssets software.

In particular, miners **do not verify the DigiAsset aspects of transactions**. Therefore, any DigiAssets client **must verify DigiAsset transactions on its own**, and to that effect must keep extra relevant data about the history of transactions.

Thus, SPV support for DigiAssets is subtle and the differences between color aware SPV clients depend on how much extra data must be kept, and how it is handled by the client in order to process and verify DigiAsset transactions. 

There are a few possibilities:
* The worst case scenario is keeping the full history of all transactions supported by the DigiAsset coin protocol, regardless of whether or not those transactions have anything to do with the assets in your wallet.
* Next is the option of keeping the full history of transactions relevant to assets in the wallet, regardless of whether those transactions involve addresses owned by the wallet. For example, say 100 units of an asset are issued. 50 units are sent to an address owned by the wallet and 50 to an address that is not owned by the wallet. In the case at hand the wallet will keep full data about the subgraph of transactions originating from the 50 units sent to the *other* address. 
* The least expensive option, and the one used by the new DigiAsset coin implementation is that of keeping full history only about transactions involving addresses owned by the wallet. The thin client will backtrack through blocks all the way to each asset issuance and keep only that data. In other words, the wallet just needs to know the subgraph of transactions that lead directly to the assets in the wallet.

# Smart Contracts capabilities

The new DigiAssets protocol implementation includes a **[Rule Engine](Rules)** that operates on rules specified in the [metadata](Metadata) of an asset.
Rules may be [Open or Locked](Rules#inheritance), when Open any one issuing or transferring the asset may add to the rule-set, when Locked the rule becomes immutable.

# Issuer Verification capabilities
Issuer verification is the process of linking a digital asset to the real world identity that issued it. 
DigiAssets based assets obtain value through a real-world promise by the asset issuers that they are willing to redeem the digital token for something of value in the real world.
Users are likely to assign value to digital tokens that can be proved to have been issued by a reputable company or individual. Verifying asset authenticity is thus at the heart of the value proposition offered by the DigiAssets protocol.

The DigiAssets protocol [supports](Static%20Data#issuer-verification) three methods of issuer verification:
* **Social**: Tying into the issuer's social graph
* **Domain**: Placing a file on an _SSL certified_ server
* **Signed**: Signing a message with a _certified_ Private-Public Key pair

Currently, only the first two possibilities are [implemented](Asset%20Verification).

If you want to read more about the subject and about how we came to the decision of supporting these verification methods, please take a look at our Asset Verification [Whitepaper](https://docs.google.com/document/d/1NYEYGI7oCRCjtbxRTLXes7M_-PeOwBF_SmwgmV1z78k)