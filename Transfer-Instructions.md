The new DigiAssets protocol defines a mini scripting language to efficiently encode the flow of assets from inputs to outputs.
Each Transfers instruction is built out of 5 pieces of data (3 booleans and two integers):

## Transfer Instructions

|Command            |Type   |Memory |Meaning|
|:-----------------:|-------|:---:|-------|
|[Skip](#skip)      |Boolean|1 bit  |0 => Stay on input after processing<br/>1 => skip to next input after processing|
|[Range](#range) *    |Boolean|1 bit   |0 => **Output** size is 5 bits and understood literally<br/>1 => **Output** size is 13 bits and understood as specifying a range|
|[Percent](#percent)|Boolean|1 bit   |0 => **Amount** is understood literally<br/>1 => **Amount** is understood as percent (1 byte)
|[Output](#output) *  |Integer|5 bits (Range=0)<hr/>13 bits (Range=1)| Specific Output index between 0..31<hr/>(**Range**=1) Range of outputs between index 0 and the specified index (maximum 8191)
|[Amount](#amount)|Integer|[1-7](Number Encoding) bytes (Percent=0)<hr/>1 Byte (Percent=1)| Number of units to be transferred <hr/> Percent of units to be transferred
&#42; In a **Burn** transaction case: *Range* = 0 and *Output* = 31 are interpreted as a burn instruction.


### Skip
Decides whether the **next** instruction will be processed against that same input (skip = 0) or against the next input (skip = 1).

### Range 
Decides whether the [Output](#output) integer specifies the actual index of an output or the maximal index in a range of outputs (starting from index 0) as the target for the transfer instruction.

&#42; Not yet supported in current version (0x02)

### Percent
Decides whether the [Amount](#amount) integer should be understood as a number of units to be transferred (percent = 0) or as a percent of existing units to be transferred (percent = 1).

&#42; Not yet supported in current version (0x02)

### Output
Specify the target output (or range of outputs) of the transfer.

* Range = 0: Make the transfer to the output of the specified index. 
Uses 5 bits, thus allowing to specify any output index between 0 and 31. 
* Range = 1: Make the **same transfer** to **all outputs** up to the specified index.
Uses 13 bits, thus allowing to specify output ranges as big as 0-8191. Useful for Mass activities such as a sending a Christmas present coupon to all the employees in a company.  

### Amount
Specifies the amount of units (or percentage of units if percent = 1) to be transferred. 
* Percent = 0: Amount is a signed integer encoded in our [precision encoding scheme](Number Encoding) and requiring 1-7 bytes of memory, depending on the number.
* Percent = 1: The integer represents percents (with some precision after the decimal point).

#### Note on Memory Consumption
Note that the total memory requirement for encoding the Skip+Range+Percent+Output of each transfer instruction is exactly 1 or 2 bytes (depending on whether Range is 0 or 1). Therefore, each transfer instruction requires between 2 to 9 bytes, depending on the Amount.

## Transfer Rules

The rules by which the DigiAssets software transfers assets between inputs and outputs when parsing transfer instructions are as follows:
* Begin with the first asset of the first input in the transaction (order is determined recursively) 
* Transfer Instruction validation rules:
  * The currently processed input must be an existing index.
  * *Output* must indicate an existing index (irrelevant in burn case).
  * *Amount* must not exceed the amount of the currently processed asset, unless: the asset is *aggregatable*, a next asset with the same asset ID exists and with such amount which can satisfy the remaining instruction amount.<br>
  * In case of an invalid transfer instruction, all assets are sent to the last output.
* Range
  * **0** move the specified amount of the currently processed asset to the output in the index specified in *Output*. The maximal index is 31.
  * **1** move the specified amount of the currently processed asset to all outputs in the range of indices from 0 all the way to the index specified in *Output*. The maximal index is 8191.
* Burn<br>
  * Parsing a **Burn** transaction (indicated by using a burn [OP_CODE](OP_CODEs) as part of the [DigiAsset scheme](DigiAsset Scheme)): *Range* = 0 and *Output* = 31 (together) are interpreted as a burn instruction.<br>In that case - the specified amount is reduced from the processed asset and is not sent to any output.

* All assets with remaining amounts are automatically transferred to the last output.

## DigiByte requirements
* A minimal dust amount in real satoshis is sent to each output for validity
* A transaction fee is left for miners to finance the processing of the transaction

## Examples
* We have 3 assets of types `A, B, C`
* The first input in our transaction has **10** units of asset `A` and **12** units of asset `B`
* The second input has **100** units of asset `C`

The following chain of instructions
```
 0 0 0 0 3    0 0 0 1 7    0 0 1 1 50    1 0 0 2 3    0 1 0 49 2
```
means:
 
1. `0 0 0 0 3` Transfer 3 units of asset `A` from the first input to the first output (index 0). **Stay** on first input.
1. `0 0 0 1 7` Transfer the remaining 7 units of asset `A` from the first input to the second output (index 1). **Stay** on first input.
1. `0 0 1 1 50` Transfer 6 units of asset `B` to second output. 6 units because 6 is 50% of 12 and we are now processing asset `B` because the previous instructions exhausted asset `A` in the first input. **Stay** on first input.
1. `1 0 0 2 3` Transfer 3 more units of asset `B` to the third output (index 2). **Skip** to second input.
1. `0 1 0 49 2` Transfer 2 units of asset `C` from the second input to each output, starting from the first output all the way to the 50th output (index 49).
1. 3 units of asset `B` are implicitly transferred to the last output, since it had 12 units and only 9 were transferred explicitly.