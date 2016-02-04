NAR (New ARchiver)
==================

Author: Nicolas DI PRIMA, Vincent HANQUEZ

Introduction
------------

What do we solve:

* simple format for packing data with simple length specification for filepath and content.
* integrate compression, encryption, authenticated data and signature.

Requirements
------------

* indexable / mmap friendly
* no filepath size limitation (within reason !)
* Word64 for length (2 exbibytes maximum per item)
* 64 bits little endian friendly, no per byte access, every fields aligned to 64 bits.


Metadata
--------

NAR Header
----------

Size : 64 bytes (8 Word64) - unused bytes set to 0

| Field num | Index in bits | Size | Description |
| --------- | ------------- | ---- | ----------- |
| 0         | 0             | u64  | magic signature: ASCII string “[ NARH ]” |
| 1         | 64            | u64  | format version (upper 32 bits) + software version (lower 32 bits) |
| 2         | 128           | u64  | cipher type. 0 no cipher |
| 3         | 192           | u64  | compression type. 0 no compression. |
| 4         | 256           | u64  | signature position (0 == no signature header) |
| 5         | 320           | u64  | index position (0 == no index header) |
| 6         | 384           | 2 u64 | unused (0) |

Item Header
-----------

Every item in a NAR archive need to start by an item header. The header is 256bits (4 Word64, 32 bytes) and is composed of:

| Field Num | Index in bits | Size | Description |
| --------- | ------------- | ---- | ----------- |
| 0         | 0             | u64  | item header signature (usually ASCII string) |
| 1         | 64            | u64  | flags    |
| 2         | 128           | u64  | length 1 |
| 3         | 192           | u64  | length 2 |

The header is directly followed by the item payload. The next item header can be found at offset:

    current item header position + 32 bytes (item header size) + ROUND_UP64(length1) + ROUND_UP64(length2)

Note: The previous formula make sure that every item header is 64 bits aligned.

Specific Item Header
--------------------

A file item start by a item header where the item values are / represent:

* item header signature: "[ FILE ]"
* flags: bitfields of
  * bit 0 : executable
  * bit 1 : compression
  * bit 2 : ciphering
  * bit 3-63 : unused (need to be 0)
* length 1: filepath length
* length 2: file length (after compression, and after ciphering. i.e. size of the blob)

Then the item data are:

| Field      | Index in bytes                | Size in bytes                 |
| ---------- | ----------------------------- | ----------------------------- |
| filepath   | 0                             | length1                       |
| padding    | length1                       | ROUND_UP64(length1) - length1 |
| file data  | ROUND_UP64(length1)           | length2                       |
| padding    | ROUND_UP64(length1) + length2 | ROUND_UP64(length2) - length2 |


