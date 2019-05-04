# Number Encoding Scheme

## Introduction
The new DigiAssets implementation uses a precision encoding scheme for encoding the integral amount of an asset's units during issuance and transfer.
This scheme supports numbers with up to **16 significant digits** and favors **round** and **small** numbers, since it is anticipated that in most use cases an asset will be issued (or reissued) in either small or round number amounts.

The DigiAssets protocol will only use numbers with up to 15 significant digits as are customary supported by the [Double-precision](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) floating-point format.


## Technical Details

Specifically, the amount of bytes required to store an integer depends on the Mantissa and the Exponent, i.e. a number without trailing zeros (Mantissa) and exponent required to express this number in [scientific notation](https://en.wikipedia.org/wiki/Scientific_notation)
```
Number = Mantissa X 10^(Exponent)
```
For example 
* The number `300 million` ==>  `3x10^8` 
* The number `333 million` ==>  `333x10^6` 
* The number `300 million and one` ==> `300000001` (no exponent). 

In this scheme 
* Any integer in the range `0-10,000,000,000,000,000 (10^16)` can be represented with full precision. 
* The least amount of memory (1 byte) is used when storing small numbers (0..31) 
* Next in line (requiring 2 bytes) are round numbers, namely numbers that require no more than 2 non zero digits to express with full precision (e.g. `10^16` or 33 trillion or 99 but not 666).

Given a specific breakdown of bits for storing the Mantissa and bits for storing the Exponent the largest number that can be represented is
```
Number=2^(Mantissa bits-1) X 10^(2^(Exponent bits-1))
```
In the following table we list the amount of bytes used to store a number based on the number of significant digits required to express it in full precision.

| Bytes  | Max Precision Digits |Mantissa Bits|Exponent Bits| 
| :----: |:--------------------:|:---------:|:-----------:|
| 1      | 0-31*                | 5         | 0           | 
| 2      | 2                    | 9         | 4           |
| 3      | 5                    | 17        | 4           |
| 4      | 7                    | 25        | 4           |
| 5      | 10                   | 34        | 3           |
| 6      | 12                   | 42        | 3           | 
| 7      | 16                   | 54**      | 0           |


```
* These are actual numbers, not precision digits.
** Since we only used 7 options we have an unused bit in the byte that encodes the number of bytes. We added that extra bit to the Mantissa bits, thus getting to a total of 54 bits in this case.
```
The derivation of this table is straightforward (you can use [these few lines of ruby code](https://gist.github.com/assafshomer/88053ae92db4446f23a1) to reproduce it)

Basically the logic is the following. Given a number of bytes:
* With **1 byte** only use the mantissa (no encoding of the number of bytes and no exponent) so we can express all numbers in the range `0..2^5-1`
* For bigger numbers: 2-3 bits are reserved for encoding the number of available bytes. 
  When using 2-6 bytes, the number of bytes is encoded with 3 bits.<br>
  When using 7 bytes, the number of bytes is encoded with 2 bits (see table above).
* We increment the number of bits in the mantissa until we can no longer express numbers with the desired precision
* The remaining bits go to the exponent
* When the mantissa exhaust our desired precision we leave nothing for the exponent

For example, let's derive the second line of the table. 

We have 2 bytes which are 16 bits. 3 are reserved for the bytes flag so we have 13 bits left. Now we look for the smallest number of Mantissa bits such that `Mantissa X 10^(Exponent)` can still be expressed with at most 16 digits of precision but if we add one more bit to the Mantissa we can no longer support such precision. 

This number turns out to be 9, because with **9** bits for the Mantissa we have 4 bits left for the exponent. This means that the maximal number we can reach in this form is `2^9 X 10^(2^(13-9))=512 X 10^16` which is **bigger** than `10^16` (our desired precision). But if we try to add one more bit to the mantissa we have 10 bits for the mantissa and only 3 left for the exponent and so the maximal number that can be represented is `2^10 X 10^(2^(13-10))=1024 X 10^8` which is **smaller** than `10^16`, and hence doesn't support our desired precision.
 