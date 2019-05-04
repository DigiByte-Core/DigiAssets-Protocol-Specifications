# Metadata and Torrents

The new DigiAssets protocol supports adding (potentially arbitrary amounts of) metadata to any DigiAsset transaction ([issuance](DigiAsset Scheme#issuance-transaction-encoding) or [transfer](DigiAsset Scheme#transfer-transaction-encoding)). The metadata is in plain JSON format and it's inclusion is **optional**. 

When metadata is included it is not stored directly on the blockchain but rather using the [BitTorrent Protocol](https://en.wikipedia.org/wiki/BitTorrent) which gives a decentralized way to share and store data. We use the new [DHT](http://www.bittorrent.org/beps/bep_0005.html) **trackerless** torrents that remove the need for a centralized host for torrent file trackers, further supporting decentralization of our protocol.

The BitTorrent protocol uses a SHA1 hash to reference the file that stores the actual data. This 20 byte SHA1 **info_hash** is recorded on the blockchain.

## BitTorrent, SHA1 and Security
Because SHA1 hashes are no longer considered cryptographically secure this opens up the (somewhat theoretical) possibility that a preimage attack on the torrent info_hash can cause us to load the wrong data from torrents. We provide a simple counter measure to deal with this possibility. The protocol supports recording an additional SHA-256 hash of the metadata on the blockchain. This way we can verify the torrent data integrity before using it. Including the SHA-256 hash of the metadata is optional and metadata recorded on the blockchain without the additional SHA2 for verification can be considered to be of lower security.

## Torrent Blockchain Record
The torrent 20 byte SHA1 info_hash referencing the metadata in a DigiAsset transaction is currently recorded on the blockchain using a single [OP_RETURN](DigiAsset Scheme#op_return) output (if 80 bytes are enough) or potentially with one additional [(1|2) or (1|3) multisignature output](DigiAsset Scheme#multisignature-addresses-multisig), depending on the [DigiAsset scheme](DigiAsset%20Scheme#data-recording-logic) and whether the SHA-256 metadata hash is included for additional security.

### Torrents and 80 bytes OP_RETURN
Now that the vast majority of the DigiByte network is accepting **80** bytes OP_RETURN, we may include **both** the SHA1 torrent info_hash and the SHA-256 metadata hash after the OP_RETURN. In that case it may be beneficial to further compress the SHA-256 using an additional RIPEMD-160 so that also the SHA2 blockchain record size is limited to 20 bytes.

# Metadata Structure
Metadata support is one of the new novelties introduced by the new DigiAssets protocol.
There are two basic parts: [Static Data](Static Data) and [Rules](Rules), each discussed in detail in the linked sections.
Generally, the data section consists of various static data fields while the rules section encodes an extra layer of logic supported by our [rule engine](Rules) that unlocks smart contracts functionality to DigiAssets.

## Syntax

[Static Data](Static Data) goes under the **metadata** key, [Rules](Rules) go under the **Rules** key.

<pre>
{
  <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data" target="_blank">metadata</a>: {...Static data goes here...},
  <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Rules" target="_blank">rules</a>: {...Rule definitions go here...}
}
</pre>

The general metadata syntax is the following:

<pre>
{
  <b>metadata</b>: {
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#assetid" target="_blank">assetId</a>: String,
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#assetname" target="_blank">assetName</a>: String,
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#assetname" target="_blank">assetGenesis</a>: String,
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#issuer" target="_blank">issuer</a>: String,
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#description" target="_blank">description</a>: String,
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#urls" target="_blank">urls</a>: [
       {name: String, url: String, mimeType: String, dataHash: String },
       {name: String, url: String, mimeType: String, dataHash: String },
       ...
    ], 
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#userdata" target="_blank">userData</a> : {
      <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#meta" target="_blank">meta</a>: [
        {key: String, value: String, type: String},
        {key: String, value: String, type: String},
        ...
      ],
      user_key: user_value,
      user_key: user_value,
      ...
     },
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#metadata-encryption" target="_blank">encryptions</a>: [
        {key: "user_key", pubKey: 'RSA Public Key',format:'pem|der',type:'pkcs1|pkcs8' },
        ...
    ],
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Static%20Data#issuer-verification" target="_blank">verifications</a>: {
      <b>social</b>:{
        <b>network_1</b>:{ 
          ....
        },
        <b>network_2</b>:{ 
          ....
        },
       ....
     },
      <b>domain</b>:{
        <b>url</b>:"https://www.example.com/path/to/file/filename.txt"
      },
      <b>signed</b>:{
        <b>message</b>: "We at ... verifying issuance of DigiAssets asset with asset ID [...].",
        <b>signed_message</b>: "-----BEGIN CMS-----...-----END CMS-----",
        <b>cert</b>: "-----BEGIN CERTIFICATE-----...-----END CERTIFICATE-----"
      }
    }
  },
  <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Rules" target="_blank">rules</a>: {
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Rules#fees" target="_blank">fees</a>: [{}],
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Rules#expiration" target="_blank">expiration</a>: {},
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Rules#minters" target="_blank">minters</a>: [{}],
    <a href="https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/wiki/Rules#holders" target="_blank">holders</a>: [{}]	
  }
}
</pre>
All the above keys are optional (except for `assetGenesis` in case of [re-issuance by minters](Rules#Minters)).

Currently, only the hyperlinked keys are recognized by the DigiAssets code.
However, even though the basic DigiAssets code will ignore other keys, any one can fork the DigiAssets [opensource code](https://github.com/DigiAsset-coins) and add supporting logic for more keys.

## Example
<pre>
metadata: {
  <b>assetId</b>:'LDJMbzwCBWhrrXpKS7TrCfoAWYgXQhwZg1G6R',
  <b>assetName</b>: "Time Machine",        
  <b>issuer</b>: "Dr. Emmet Brown", 
  <b>description</b>: "The flux capacitor will send us back to the future",
  <b>urls</b>: [
    {
      <b>name</b>: "imdb",
      <b>url</b>: "http://www.imdb.com/title/tt0088763/",
      <b>mimeType</b>: "text/html",
      <b>dataHash</b>: "249e3e3c77d07d8fe8984a47bbbab8c89aeb8b1dadf4e2ff47db42a3e5a1c126"
    }
  ],
  <b>encryptions</b>: [
    {
      <b>key</b>: "Undelrying Physics",
      <b>pubKey</b>: "-----BEGIN RSA PUBLIC KEY-----\nMIIBCgKCAQEA0hw6PRO9RpHRf/pdpEMfD01odzBTaheuA1JxunVTq+/X1hGSUrpRWMIM/tp8\n9DQod6K+6Bo/2CmoZxkWPOk45tbU9QE4Cb532n+MIkzsmbvmM+i49UXSqC8v44MGKTVLb7X2\nPogItSM3lqH4KpZR3cM/JDarfS1R77U/OMDZ/YECDPbcwKPdSLQHhWJ1c9cX5+0lCSDt1WXY\n4XX+hH64C+L/Ss4dMP2kpyHvbsBYpGdLu7AmcDmHtCOl2rXR1z4E0asYGiojw3PI56ATOndS\n30ABKKgQTAExjPQ24BtJYhfJ+zD5zHhztizPPfOwrID2HTfGwVTwfXinV4bpoFfwhwIDAQAB\n-----END RSA PUBLIC KEY-----\n",
      <b>format</b>: "pem",
      <b>type</b>: "pkcs1"
    }
  ],
  <b>verifications</b>: {
    <b>social</b>:{
      <b>twitter</b>:{
        <b>username</b>: 'my_username'
      },
      <b>facebook</b>:{
        <b>page_id</b>: '1232952150'
      },
      <b>github</b>:{       
        <b>gist_id</b>: '6c704f5759927212e714'
      }
    },
    <b>domain</b>:{
      <b>url</b>:"https://www.example.com/digital_assets/assets.txt"
    },
    <b>signed</b>:{
      <b>message</b>: "We at example.com verifying issuance of DigiAssets asset with asset ID [LJEC6Q2h9JKNvZqEC87TbEXvxm4br1uivb2QX].",
      <b>signed_message</b>: "-----BEGIN CMS-----
MIIFawYJKoZIhvcNAQcCoIIFXDCCBVgCAQExDzANBglghkgBZQMEAgEFADCBgQYJ
KoZIhvcNAQcBoHQEcldlIGF0IGV4YW1wbGUuY29tIHZlcmlmeWluZyBpc3N1YW5j
ZSBvZiBjb2xvcmVkIGNvaW5zIGFzc2V0IHdpdGggYXNzZXQgSUQgW0xKRUM2UTJo
OUpLTnZacUVDODdUYkVYdnhtNGJyMXVpdmIyUVhdLqCCAwMwggL/MIIB56ADAgEC
AgEBMA0GCSqGSIb3DQEBBQUAMBoxCzAJBgNVBAYTAlVTMQswCQYDVQQKDAJaNDAe
Fw0xMzA4MjgxODI4MzRaFw0yMzA4MjgxODI4MzRaMBoxCzAJBgNVBAYTAlVTMQsw
CQYDVQQKDAJaNDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAN906qi0
d3nlJg7R0vPahd6eDD+1n6rdmY495WYy44whg78K/XCTZTQ4rim6Dg4FIk+GXT1m
zojAHw3A85TsQNOAw5xBRgH/pA0IjUntmbagu25yHPPikhv6jhkCFWmsuFHK+qLw
5MpDuR8Z+zlO7CafUz/R9CR/MzEAOmM4o2B/S7LqU3q62N1Os8ooVRA11zr9PLwR
5OdATBSWxPYsBcJ8QrFOflGVGWMPO1LtJ+CzxUvQU/LVCLwH50VyBFgoWtymxVpn
WUGZcqxcNG7lBH9GDd/0NGrgZHWw0fxEtj24CkyXZI3P6xHjNE8bzlH+x7lDToa7
QFRDdQ+brsRNVksCAwEAAaNQME4wHQYDVR0OBBYEFAdk9MTJBW7/Qj/dBaGVPJbt
ahPNMB8GA1UdIwQYMBaAFAdk9MTJBW7/Qj/dBaGVPJbtahPNMAwGA1UdEwQFMAMB
Af8wDQYJKoZIhvcNAQEFBQADggEBAMfGon3LxqVMftqBUxZl+Jz9Fut28fDKr4g6
uANOirKjTD4hnoEKf2+O/mK6Gq7gWis5YMAWlAplbQMsEkRDL/v/T72mTYM+ErVf
N06i+VKuRG34ZwSkadRefMAJcMFs7T5auT6FyMSRBFErVtbauScBEgQkb0ZL6lKE
/0Gr/QGY2wx6l01wFrTPdrkHR/MXIJChSVfpFOrFHDWHVC3kXTSl+yntKNdUi9hx
7Ado0BJu2jWfmGtLPVVj2EttrXWK8vU3hahZubMAvoFZpuwLYP8x6dJPb9fqFMrI
ZemDcSDaHeIu9S3Bw2fLSFEqy1zvcmX3FHepd9cokfROFOWrSCoxggG1MIIBsQIB
ATAfMBoxCzAJBgNVBAYTAlVTMQswCQYDVQQKDAJaNAIBATANBglghkgBZQMEAgEF
AKBpMBgGCSqGSIb3DQEJAzELBgkqhkiG9w0BBwEwHAYJKoZIhvcNAQkFMQ8XDTE1
MTAxMzA4MjM0N1owLwYJKoZIhvcNAQkEMSIEIEmnhXUqo76ePeBF1RGvV0oCDsGZ
3kF+Z6RvD3eC8lSXMA0GCSqGSIb3DQEBCwUABIIBADWkvjI5Adtlj7MKZ6m8Q20Y
uonGMQBmOJNBIfG2X+nfRFO7FGKT7hGT7Hx5Hkx2oStljdMC1C/orNQUzC7BORuF
JKLktO79a4nSpegEN5X1JK2ZLsifUONup3EgLGNj9NR5AIGlLM7fLVKAAPrOLnlf
mXt5awcgLEztzM+/W7HMPDS+neKGwFtDdjZgNOQjGKehnjWv8vDeslIxVq+FHlAy
0GYd2y14RwTxejIU1mpXRP6I1J8/9OwUNmW8J1oELi9qgx/WdeppXwZma9QLWBU7
qZT5uAGldHG1UJl+sJ266J96wB5TyvCXzdT+G5Pw7as3tn1GIXel/hXIMu0Nq00=
-----END CMS-----",
     <b>cert</b>: "-----BEGIN CERTIFICATE-----
MIIC/zCCAeegAwIBAgIBATANBgkqhkiG9w0BAQUFADAaMQswCQYDVQQGEwJVUzEL
MAkGA1UECgwCWjQwHhcNMTMwODI4MTgyODM0WhcNMjMwODI4MTgyODM0WjAaMQsw
CQYDVQQGEwJVUzELMAkGA1UECgwCWjQwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAw
ggEKAoIBAQDfdOqotHd55SYO0dLz2oXengw/tZ+q3ZmOPeVmMuOMIYO/Cv1wk2U0
OK4pug4OBSJPhl09Zs6IwB8NwPOU7EDTgMOcQUYB/6QNCI1J7Zm2oLtuchzz4pIb
+o4ZAhVprLhRyvqi8OTKQ7kfGfs5Tuwmn1M/0fQkfzMxADpjOKNgf0uy6lN6utjd
TrPKKFUQNdc6/Ty8EeTnQEwUlsT2LAXCfEKxTn5RlRljDztS7Sfgs8VL0FPy1Qi8
B+dFcgRYKFrcpsVaZ1lBmXKsXDRu5QR/Rg3f9DRq4GR1sNH8RLY9uApMl2SNz+sR
4zRPG85R/se5Q06Gu0BUQ3UPm67ETVZLAgMBAAGjUDBOMB0GA1UdDgQWBBQHZPTE
yQVu/0I/3QWhlTyW7WoTzTAfBgNVHSMEGDAWgBQHZPTEyQVu/0I/3QWhlTyW7WoT
zTAMBgNVHRMEBTADAQH/MA0GCSqGSIb3DQEBBQUAA4IBAQDHxqJ9y8alTH7agVMW
Zfic/RbrdvHwyq+IOrgDToqyo0w+IZ6BCn9vjv5iuhqu4ForOWDAFpQKZW0DLBJE
Qy/7/0+9pk2DPhK1XzdOovlSrkRt+GcEpGnUXnzACXDBbO0+Wrk+hcjEkQRRK1bW
2rknARIEJG9GS+pShP9Bq/0BmNsMepdNcBa0z3a5B0fzFyCQoUlX6RTqxRw1h1Qt
5F00pfsp7SjXVIvYcewHaNASbto1n5hrSz1VY9hLba11ivL1N4WoWbmzAL6BWabs
C2D/MenST2/X6hTKyGXpg3Eg2h3iLvUtwcNny0hRKstc73Jl9xR3qXfXKJH0ThTl
q0gq
-----END CERTIFICATE-----"
    }
  },               
  <b>userData</b> :{
    <b>meta</b>: [
      {<b>key</b>: 'Weight', <b>value</b>: 50000,      <b>type</b>: 'Number'},
      {<b>key</b>: 'Model',  <b>value</b>: "Delorean", <b>type</b>: 'String'},       
    ],
    <b>technology</b>: 'flux capacitor 666',
    <b>"Undelrying Physics"</b>: 'This magnetic flux calculator calculates the magnetic flux of an object based on the magnitude of the magnetic field which the object emanates and the area of the object, according to the formula, Φ=BA, if the magnetic field is at a 90° angle (perpendicular) to the area of the object. If the magnetic field is not perpendicular to the object, then use the calculator below, which computes the magnetic flux at non-perpendicular angles. The magnetic flux is directly proportional to the magnitude of the magnetic field emanating from the object and the area of the object. The greater the magnetic field, the greater the magnetic flux. Conversely, the smaller the magnetic field, the smaller the flux. The area of the object has the same direct relationship. The greater the area of an object, the greater the flux. Conversely, the smaller the area, the smaller the magnetic flux.'
  } 
}
</pre>