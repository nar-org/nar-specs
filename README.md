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
* Word64 for length (2 exbibytes maximum per file)
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

    current item header position + 32 bytes + ROUND_UP64(length1) + ROUND_UP64(length2)

