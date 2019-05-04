This document describes DigiAssets protocol specification for issuing and transacting digital asset on top of the DigiByte Blockchain. DigiAssets is a concept that allows attaching metadata to DigiByte transactions and leveraging the DigiByte infrastructure for issuing and trading immutable digital assets that can represent real world value. Digital asset on top of DigiByte's Blockchain can be used to issue Financial assets (securities like shares, commodities like Gold or new currencies), Prove of ownership (A digital key to a house or a car, a concert ticket) or to store information (Documents, Certificates, Contracts)

#This Document is divided to 2 parts:

##The DigiAsset scheme

The DigiAsset scheme describes the encoding used to compress the metadata that is stored in the OP_RETURN operator. This DigiAsset scheme is optimized to allow robust functionality by inserting the most data within constraints of the Blockchain (80 Bytes)

##Rule engine

This document introduces the first specification for handling metadata for digital assets. Because of the 80 Bytes limit of the OP_RETURN field, any metadata that is attached to a transaction is hashed and stored in torrents to allows decentralized access to the information. By hashing the metadata for every transaction, DigiAssets actually expands dramatically the basic DigiByte functionality by allowing to insert rules that exist outside the blockchain and can be publically accessed via torrents and verified against a specific transaction thus creating a rule engine with basic smart contract functionality.