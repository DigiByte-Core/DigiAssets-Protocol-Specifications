This set of OP_CODES are only recognized by the DigiAssets software and should not be confused with [DigiByte OP_CODES](https://en.DigiByte.it/wiki/Script) (such as OP_RETURN).

DigiAssets OP_CODES are encoded using a single byte. Half of the values represented by that byte are reserved for Core DigiAssets protocol OP_CODES, the other half is left open for custom OP_CODES that may be added by users (e.g. in forks of the DigiAssets opensource software).

In particular
* The first 128 values `0x00-0x7F` are reserved for DigiAssets OP_CODES
 * `0x01-0x0F` represent **Issuance** OP_CODES
 * `0x10-0x1F` represent **Transfer** OP_CODES
 * `0x20-0x2F` represent **Burn** OP_CODES
* The last 128 values `0x80-0xFF` can be used for referring to new OP_CODES used by custom extensions (plugins) 

The table below lists the (currently available) DigiAssets OP_CODES 
* The **I**/**T**/**B** column refers to whether this is an [Issuance](DigiAsset%20Scheme#issuance-transaction-encoding), a [Transfer](DigiAsset%20Scheme#transfer-transaction-encoding) or a [Burn](DigiAsset%20Scheme#burn-transaction-encoding) OP_CODE.
* The **M** column designates whether any [Metadata](Metadata) was stored at all 
* The **1(2)** column designates whether a (1\|**2**) multisignature address was used in addition to OP_RETURN
* The **1(3)** column designates whether a (1\|**3**) multisignature address was used in addition to OP_RETURN

| Hex   |Meaning|I/T/B|M|1(2)|1(3)|Comment|
| :---: |-------|---|:------:|:----:|:-----:|-------|
| `0x00`            | Undefined
| `0x01`            | SHA1 Torrent info_hash + SHA256 of metadata in **80** bytes OP_RETURN| I | &#10004; | &#10005; | &#10005; |  |
| `0x02`            | SHA1 torrent info_hash in OP_RETURN<br/>SHA256 of metadata in 1(**2**) multisig| I| &#10004; | &#10004; |&#10005;|
| `0x03`            | SHA1 Torrent info_hash + SHA256 of metadata in 1(**3**) multisig |I|&#10004; |&#10005;|&#10004;|
| `0x04`            | SHA1 Torrent Hash in OP_RETURN<br/>No SHA256 of metadata|I|&#10004;|&#10005;|&#10005;|Low Security
| `0x05`            | No Metadata, cannot add [rules](Rules)| I|&#10005;|&#10005;|&#10005;|Locked
| `0x06`            | No Metadata, can add [rules](Rules)| I|&#10005;|&#10005;|&#10005;|Unlocked
| `0x10`            | SHA1 Torrent info_hash + SHA256 of metadata in **80** byte OP_RETURN|T| &#10004; | &#10005; |&#10005; | |
| `0x11`            | SHA256 of metadata in 1(**2**) multisig|T| &#10004; |&#10004;|&#10005;|
| `0x12`            | SHA1 Torrent info_hash + SHA256 of metadata in 1(**3**) multisig |T|&#10004; |&#10005;|&#10004;||
| `0x13`            | SHA1 Torrent info_hash in OP_RETURN<br/>No SHA256 of metadata|T|&#10004;|&#10005;|&#10005;||Low Security
| `0x14`            | SHA1 Torrent info_hash in OP_RETURN<br/>No SHA256 of metadata<br/>No rules in metadata|T|&#10004;|&#10005;|&#10005;|Low Security, Ignore rules if exist
| `0x15`            | No Metadata|T|&#10005;|&#10005;|&#10005;|
| `0x20`            | SHA1 Torrent info_hash + SHA256 of metadata in **80** byte OP_RETURN|B| &#10004; | &#10005; |&#10005; | |
| `0x21`            | SHA256 of metadata in 1(**2**) multisig|B| &#10004; |&#10004;|&#10005;|
| `0x22`            | SHA1 Torrent info_hash + SHA256 of metadata in 1(**3**) multisig |B|&#10004; |&#10005;|&#10004;||
| `0x23`            | SHA1 Torrent info_hash in OP_RETURN<br/>No SHA256 of metadata|B|&#10004;|&#10005;|&#10005;||Low Security
| `0x24`            | SHA1 Torrent info_hash in OP_RETURN<br/>No SHA256 of metadata<br/>No rules in metadata|B|&#10004;|&#10005;|&#10005;|Low Security, Ignore rules if exist
| `0x25`            | No Metadata|B|&#10005;|&#10005;|&#10005;|