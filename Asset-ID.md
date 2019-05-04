The asset ID is an immutable identifier of an asset that is created at issuance and is derived from the issuance transaction and the asset's [divisibility](DigiAsset-Scheme#asset-divisibility) and [aggregation policy](DigiAsset-Scheme#asset-aggregation-policy). 

There are two types of Asset IDs corresponding to the two types of assets supported by the new DigiAssets protocol: [locked](Benefits#locked-assets) and [unlocked](Benefits#unlocked-assets).

## Unlocked Asset IDs
Unlocked asset IDs look similar to DigiByte addresses, except instead of starting with the number **1** start with the capital letter **U**.

An unlocked Asset ID is derived from the [pubkey script](#example) of the UTXO referenced in the **first input** of the issuance transaction similar to the way that a [DigiByte address is derived from a public key](https://en.DigiByte.it/wiki/Technical_background_of_version_1_DigiByte_addresses).
The difference is that instead of padding the network version `0x00` which gives the prefix 1 to standard DigiByte addresses, we pad with a pre-defined 2 bytes sequence which ensures the asset ID has the prefix `U`, followed by one of the characters `a`, `h` or `d`, depending on the asset [aggregation policy](#aggregation_policy):

| Aggregation policy | Padding |
|--------------------|:-------:|
| aggregatable       | 0x2e37  |
| hybrid             | 0x2e6b  |
| dispersed          | 0x2e4e  |

In detail, the process is
```
script = pubkey script of the output referenced in first input of issuance transaction
padding = 2 bytes according to the asset aggregation policy
hash160 = RIPEMD-160(SHA-256(script))
divisibility = asset divisibility extended to 2 bytes
Unlocked Asset ID = Base58Check(padding+hash160+divisibility)
```
where `+` means concatenation.

An example of such an asset ID is 
```
Ua9kP82bbaxtYodahVBY8zs6cZ78Xua4X97Nu8
```

## Locked Asset IDs
Locked asset IDs look similar to DigiByte addresses, except instead of starting with the number **1** start with the capital letter **L**.

A Locked Asset ID is derived in a similar way, except instead of using the script of the referenced output, we use the [transaction ID + index](#example) of the UTXO referenced in the **first input** of the issuance transaction.
For padding, we use a different table of 2 byte sequences, which ensures the asset ID has the prefix `L`, followed by one of the characters `a`, `h` or `d`, depending on the asset [aggregation policy](DigiAsset-Scheme#asset-aggregation-policy):

| Aggregation policy | Padding |
|--------------------|:-------:|
| aggregatable       | 0x20ce  |
| hybrid             | 0x2102  |
| dispersed          | 0x20e4  |

In detail, the process is
```
txid = Transaction ID of first UTXO in the issuance transaction 
idx = Index of first UTXO in the issuance transaction 
padding = 2 bytes according to the asset aggregation policy
hash160 = RIPEMD-160(SHA-256(txid+':'+idx))
divisibility = asset divisibility extended to 2 bytes
Locked Asset ID = Base58Check(padding+hash160+divisibility)
```
where as before `+` means concatenation.

An example of such an asset ID is 
```
Ld9ohgk6wruYptPeoDQRHRif8fG2vJe1uN4ryz
```
## Comments

### Asset IDs are divisibility and aggregation-policy dependent
The reason the asset [divisibility](DigiAsset-Scheme#asset-divisibility) and [aggregation-policy](DigiAsset-Scheme#asset-aggregation-policy) is included in the asset ID is to prevent the edge case of issuing the same asset twice, **each time with a different divisibility or aggregation policy**.
For technical reasons (basically in order to get the correct **L**,**U** prefixes) we had to extend the [3 bits defining the divisibility](DigiAsset-Scheme#issuance-flag) to 2 full bytes by padding them with zeros.

### Asset IDs and DigiByte Confirmations
Note that both asset ID types make no reference to a DigiByte block and hence are meaningful even at zero confirmations.
Thanks to that, and to our [DigiAsset scheme](DigiAsset Scheme) we can perform [issuance and transfer](DigiAsset Scheme#issuance-transaction-encoding) of an asset in the same transaction.

### Unlocked
Since an unlocked asset ID uses a transaction output pubkey script, all assets that originate from an output carrying the exact same pubkey script, are considered the same. For example, when the issuance transaction's first input points to a standard pay-to-public-key-hash output, we can perform more than one issuance transactions (provided we own the corresponding private key) and issue more units of that asset by using outputs payed to the same address, since they carry the exact same pubkey script, which is why it is unlocked.

### Locked
Since locked asset IDs reference a transaction id, any attempt to issue more of that asset will necessarily generate a new asset ID, so this specific asset can only be issued once, which is why it is locked.

### Lock status and the [Rules Engine](Rules Engine)
Since the new DigiAssets protocol supports an extra layers of rules it is possible to override the locked status by declaring [Minters](Rules#minters) in the [Metadata](Metadata)

### Example
The data fields relevant for the construction of an asset id are the **transaction ID** and **index** of the UTXO in case of a [locked asset](#locked) or the **pubkey script** of the UTXO in case of a [unlocked asset](#unlocked).

As an example, look at the following [transaction](http://DigiAssets.org/explorer/testnet/tx/813463168a42b117e14c80854b71f5c1d7824d96e092c6648a5adfd49acd7e4b). If we were to build a locked asset id we would be interested in the transaction ID and index corresponding to the fields `prev_out.hash` and `prev_out.n` below.
If on the other hand we were to build an unlocked asset id, we would need the pubkey script of that UTXO. In most cases, including the most common type of a payment as in this example, pay-to-public-key-hash, we can construct that pubkey script by using data from the issuance transaction itself without needing to fetch the referenced UTXO.
Specifically, we would locate the public key of the credited address in the second part of the `scriptSig`.
Then, we can construct and encode the predicted pubkey script.

<pre>
{
    "hash": "813463168a42b117e14c80854b71f5c1d7824d96e092c6648a5adfd49acd7e4b",
    "ver": 1,
    "vin_sz": 1,
    "vout_sz": 3,
    "lock_time": 0,
    "size": 297,
    "in": [
        {
            "prev_out": {
                "hash": "<b>62cd78e2859a2b7ad1f0a08d855ae881073f5b7beb3e98957c22d5ee2f081932</b>",
                "n": <b>0</b>
            },
            "scriptSig": "3045022100b5aaae72b05c0698ea22e2f4cb3f3a46e5a0a1c1a98772b1c7305476b9ae5e1f02200276a003694eab8d12bc5791624b60b1c68486e4b985f2a672751bb35295202b01 <b>02b509613c7e5d9e47347635f872c3aa271d01ac4a9a6445839ce2c5820a0f48a8</b>",
            "sequence": 4294967295
        }
    ],
    "out": [
        {
            "value": "0.00000600",
            "scriptPubKey": "OP_DUP OP_HASH160 97d2ee97e28c1cdb1c9b7febdf7c028062e99fed OP_EQUALVERIFY OP_CHECKSIG"
        },
        {
            "value": "0.00000000",
            "scriptPubKey": "OP_RETURN 43430201d36854c461e1ce4d90dd73857ae9fa5349f6ba4ada9f9603314dcbc0225f39b9313b7d67a5059e00e0b179e01cf8f879eaaa7fa31a001a48",
            "DigiAssetCoinsData": {
                "payments": [
                    {
                        "input": 0,
                        "amount": 26,
                        "output": 0,
                        "range": false,
                        "precent": false
                    }
                ],
                "protocol": 17219,
                "version": 2,
                "type": "issuance",
                "lockStatus": false,
                "aggregationPolicy": "dispersed",
                "divisibility": 2,
                "amount": 26,
                "multiSig": [],
                "torrentHash": "d36854c461e1ce4d90dd73857ae9fa5349f6ba4a",
                "sha2": "da9f9603314dcbc0225f39b9313b7d67a5059e00e0b179e01cf8f879eaaa7fa3"
            }
        },
        {
            "value": "0.00000600",
            "scriptPubKey": "OP_DUP OP_HASH160 97d2ee97e28c1cdb1c9b7febdf7c028062e99fed OP_EQUALVERIFY OP_CHECKSIG"
        }
    ]
}
</pre>

In this example JSON we can see the `DigiAssetCoinsData` decoded from the OP_RETURN:
the asset lockStatus is false, its divisibility is 2, and the aggregation policy is "dispersed".
So we apply the [unlocked asset](#unlocked) ID derivation:
```
pubkey = 0x02b509613c7e5d9e47347635f872c3aa271d01ac4a9a6445839ce2c5820a0f48a8
pubkeyhash = RIPEMD-160(SHA-256(pubkey)) = 0x97d2ee97e28c1cdb1c9b7febdf7c028062e99fed
script = hex("OP_DUP OP_HASH160 "+pubkeyhash+" OP_EQUALVERIFY OP_CHECKSIG022d18e1a4d1d97d15") = 0x76a91497d2ee97e28c1cdb1c9b7febdf7c028062e99fed88ac
padding = 0x2e4e
hash160 = RIPEMD-160(SHA-256(script)) = 0xb8d7e15197646177bfa7bd167d0d5bde826cf2bb
divisibility = 0x0002
Unlocked Asset ID = Base58Check(padding+hash160+divisibility) = "UdDszTzN2NV4frsfVeXQKDiTZMbRvMqDfK6KF4"
```
where as `+` means concatenation.