This part of the metadata includes optional static data fields. Currently the DigiAssets implementation recognizes the following keys:

## assetId
Including the [assetId](Asset ID) in the metadata during asset [transfer](DigiAsset%20Scheme#transfer-only-transactions), while optional, can expedite processing.
Note that one cannot include the Asset ID during [issuance](DigiAsset%20Scheme#issuance-and-transfer-transactions) because the Asset ID was not yet determined. 
## assetName
Asset Name as a free string
## assetGenesis
This field is required only upon re-issuance of an asset by a [minter](Rules#minters), regardless of whether the asset is [locked or unlocked](Benefits#coherent-issuance-policy). 
The genesis field stores the block and transaction ids of the issuance transaction.
## issuer
A free string describing the Asset's issuer
## description
A free string describing the Asset
## urls
An array of JSON objects, each containing a link to a file (hosted on some remote server) containing data about the asset.
```js 
{
    name: String,
    url: String,
    mimeType: String, // mime type of data in that url
    dataHash: String // SHA-256 hash of the data (to allow data verification)
}
```

the following are predefined names for use with the DigiAssets explorer:  
```icon``` - this will be the icon of the asset on asset pages and lists

```large_icon``` - a larger version of the icon to be displayed on larger screens

Other examples:

consider a legal contract describing the issuer's promise to redeem each unit for goods or services in the real world. 
That legal document file can be hosted on the issuer's company website and linked using the URL data field. A hash of the document can be stored as well to ensure that the issuer (or otherwise host of the data) did not alter it.
Both issuers and holders of an asset can thus refer unambiguously to the original contract against which the asset was issued.

## userData
A list of free JSON objects.
### meta 
A special key named `meta` is used to allow the [DigiAssets explorer](http://DigiAssets.org/explore) and client side applications to display said data.
The value of the `meta` key should be an array of `{key,value,type}` objects that will be displayed by the explorer.
```
{key: String, value: String, type: String}
```  
Data Types that are currently parsed by the explorer:  
```
String, Number, Boolean, Date, URL, Email, Array
```  

the ```Array``` type expects an array of ```{key: String, value: String, type: String}``` objects, this allows building hierarchical data formation.

if a Type is not specified the parser defaults to the JSON datatype (String, Number or Boolean)

Examples:

```
{key: "user name", value: "username", type: "String"}
{key: "birthday", value: "01/01/1970", type: "Date"}

{
    key: "company", 
    value: [
        {key: String, value: String, type: String},
        {key: String, value: String, type: String}
    ],
    type: "Array"
}
```

### free JSON
On top of the **meta** key, whose values are parsed and displayed in the [DigiAssets explorer](http://DigiAssets.org/explore), you can add arbitrary key value pairs.

The general syntax of the **userData** key is thus

```
userData : {
      meta: [
              {key: String, value: String, type: String},
              {key: String, value: String, type: String},
               .......
            ],
      user_key: user_value,
      user_key: user_value,
      ...
     },
```

## Metadata Encryption

The DigiAssets protocol supports the use of [RSA public keys](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) for metadata encryption.
Any (or all) of the [free JSON user data](Static-Data#free-json) **values** can be encrypted by specifying 
* The JSON **key** whose value is to be encrypted  
* The [format](https://tls.mbed.org/kb/cryptography/asn1-key-structures-in-der-and-pem): `pem` or `der` 
* The [padding scheme](https://en.wikipedia.org/wiki/RSA_(cryptosystem)#Padding_schemes): `pkcs1` or `pkcs8`

To encrypt user metadata add an **encryptions** key whose value is an array
<pre>
<b>encryptions</b>: 
 [
   {key: "user_key", pubKey: 'RSA Public Key',format:'pem|der',type:'pkcs1|pkcs8' },
   {key: "user_key", pubKey: 'RSA Public Key',format:'pem|der',type:'pkcs1|pkcs8' },
   .......
 ]
</pre>
each element of the array is a JSON of the form
<pre>
 {
   <b>key</b>:String, 
   <b>pubKey</b>:String, 
   <b>format</b>:String, 
   <b>type</b>:String
  }
</pre>
Where
* **key**: The [free JSON user data](Static-Data#free-json) key whose value is to be encrypted
* **pubKey**: An RSA public key
* **format**: The RSA public key [format](https://tls.mbed.org/kb/cryptography/asn1-key-structures-in-der-and-pem) (`pem` or `der`)
* **type**: The RSA [padding scheme](https://en.wikipedia.org/wiki/RSA_(cryptosystem)#Padding_schemes) ( `pkcs1` or `pkcs8`)

Here is [an example](Metadata#example).

## [Issuer Verification](Benefits#issuer-verification-capabilities)
Issuer verification is supported by adding a **verifications** key to the metadata.

<pre>
 <b>verifications</b>: {
   <b>social</b>:{
      <b>network_1</b>:{
         .........          
      },
      <b>network_2</b>:{
         .........          
      },
       ..........
   },
   <b>domain</b>:{
      <b>url</b>:"https://www.example.com/path/to/file/filename.txt"
   },
   <b>signed</b>:{
      <b>message</b>: "plain text message",
      <b>signed_message</b>: "signed message",
      <b>cert</b>: ssl certificate (in some format)
   }
}
</pre>

Each social media requires slightly different data, so for example we can have
<pre>
 <b>verifications</b>: {
   <b>social</b>:{
      <b>twitter</b>:{
         <b>username</b>: 'my_username'
      },
      <b>facebook</b>:{
         <b>page_id</b>: '1233454356'
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
}
</pre>