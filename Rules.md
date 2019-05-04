
This part of the metadata is one of novel features of the new DigiAssets protocol which opens up the possibility of creating [smart contracts](http://www.fastcolabs.com/3035723/app-economy/smart-contracts-could-be-cryptocurrencys-killer-app) with DigiAssets.

The current implementation of the DigiAssets protocol supports 4 kinds of rules:
* [Fees](#fees): Pay a fee to specific address on each transfer
* [Expiration](#expiration): The asset expires after a while
* [Minters](#minters): Recipients of the asset can issue more of it
* [Holders](#holders): Only certain DigiByte addresses can receive this asset

Each one of the above 4 rule types accepts a JSON object (or objects) with two keys
### params
Specifying relevant parameters to be discussed below
### Inheritance
Specifying how this rule is inherited upon transferring the asset. The inheritance flag can accept the numbers `0`, `1` or `2`. The inheritance rules are less strict (giving more freedom to the new recipient of the asset) as the number increases.
Roughly you should think of those numbers as designating:
* `0` : Low privilege (Locked)
* `1` : Limited privilege
* `2` : Admin (Open)

# Fees

### Description

Determines if there should be a fee when an Asset is transferred and what address should receive the fee,
there could be multiple fee rules for one asset. One can think of this rule as facilitating **Asset Affiliation**.

### Default
By default the fees rule is empty and the rule is [open](#inheritance). 

### Params
The Fees rule can accept an array of JSON objects each specifying the following parameters:
* **amount**: The amount of units of the relevant asset that must be payed to the address
* **address**: The DigiByte address to which the amount must be transferred
* **asset_id**: The [Asset ID](Asset ID) of the asset the fee should be payed with.

Note: `asset_id = 1` will be reserved for the native token (e.g. DigiBytes on the DigiByte network). 

### Inheritance

* **0** : The new recipient of the asset cannot add fees
* **1** : The new recipient of the asset can add more fees, but only of inheritance level 0
* **2** : The new recipient of the asset can add more fees without restriction

### Use case
Creating a decentralized secondary marketplace for some assets (e.g. ticketing) has a problematic business model. When creating an asset and selling it in the market the issuer looses control over the market and receives no fees when the asset is being transferred between users. 
This rule will allow companies to use the security mechanism of the blockchain to eliminate fraud while still gaining from each transaction being made outside of the companies centralized database.
 

# Expiration
### Description
Determines the lifespan of an Asset. When expiration time is reached the asset returns to the last valid Output with a valid expiration.
### Default
By default the expiration rule is empty and [open](#inheritance).  

### params
The Expiration rule can accept JSON object with two parameters:
* **blocks**: [encoded](Number Encoding) integer specifying a block-height or a range.
* **range**: A boolean specifying whether 
 * range = 0 : The block parameter is the height of a specific block that when reached the asset expires, or
 * range = 1 : The block parameter designates range `N`. The asset will expire when `N` blocks were mined on top of the block of the issuance transaction. 

### Inheritance

* **0** : The new recipient of the asset cannot change the expiration
* **1** : The new recipient of the asset can change the expiration, but only of inheritance level 0
* **2** : The new recipient of the asset can change the expiration without restriction

Note that changing the expiration can only be done subject to the original expiration. For example, if an asset with expiration time of 10 blocks and inheritance flag 2 is transferred, a new expiration time of 15 cannot make the asset live longer than dictated by the original setting of 10 blocks. Thus, changing an expiration can be only used meaningfully to shorten the expiration time.

### Use case
Time based assets could be used for loans, limited time access assets like a key, a coupon valid for X amount of time and many other use cases where an asset has a life time. 

### Example
A time sharing platform like Airbnb issuing a "key" asset that opens a specific apartment door for X amount of time.  

# Minters
### Description
Addresses that are allowed to re-Issue an Asset, regardless of the [asset type](Benefits#coherent-issuance-policy).

### Default
By default the minters rule is empty and [Locked](#inheritance).

### params
The Minter rule can accept a list of JSON object with a single parameter:
* **address**: A DigiByte address. Anyone with the private key to this address can issue more units of the asset, even if it is
 * [locked](Benefits#locked-assets)
 * [unlocked](Benefits#unlocked-assets) but the minter doesn't have the private key for the address where it was originally issued . 

### Inheritance

* **0** : The new recipient of the asset cannot add minters (asset is locked as far as the rules engine is concerned)
* **1** : The new recipient of the asset can add minters, but only of inheritance level 0
* **2** : The new recipient of the asset can add minters without restriction

Note that, for the sake of performance optimization, in case of reissuing an asset by a minter (regardless if the asset is [locked](Benefits#locked-assets) or not) the [Asset_Genesis](Metadata)  pointing to the transaction and block of the first issuance, must be provided in the metadata. 

### Use case
Give "Admin" permission to predefined addresses to issue more of the same assets or add meta data. 

### Example
Headquarters of a bank issuing a *USD coin* backed by the bank and want to give permission to other branches of the same bank (or other banks) to issue more of the same USD coins.   


# Holders

### Description
Issuing an asset with a set of rules on what addresses are allowed to hold this Asset. 

### Default
by default the holders rule is empty and Open.

### params
The Holders rule can accept a list of JSON object with a single parameter:
* **address**: A DigiByte address. The asset can only be sent to one of those addresses, so only the limited group that controls this list of addresses can hold the asset.

### Inheritance

* **0** : The new recipient of the asset cannot add holders
* **1** : The new recipient of the asset can add holders, but only of inheritance level 0
* **2** : The new recipient of the asset can add holders without restriction

### Use case
One of the biggest debates for using a permissionless blockchain (like DigiByte's) as a backbone for financial institutions is the ability to control the assets in a manner that will comply with regulations like [AML](http://www.investopedia.com/terms/a/aml.asp) and [KYC](http://www.investopedia.com/terms/k/knowyourclient.asp). 
Using this rule one would be able to issue an asset that is valid only when stored in predefined confirmed addresses, thus effectively creating a permission layer inside a permissionless blockchain. 

#### Example 
A private equity market issuing an asset that only accredited investor are allowed to buy and hold a specific address after a full KYC. In that case the exchange/ crowdfunding platform will verify the identity and the address by signing a transaction and match between the person and the address.
Sending or selling the asset to an unconfirmed address will end in loosing the asset (the asset will become invalid in an unregistered address). 
The issuer of the asset would be able to add more permissions (addresses) to the list of confirmed addresses the asset will be valid in if the flag locked is false.