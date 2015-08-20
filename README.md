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

| Index | Size | Description |
| ----- | ---- | ----------- |
| 0     | 8    | magic signature: ASCII string “[ NARH ]” |
| 8     | 8    | format version (upper 32 bits) + software version (lower 32 bits) |
| 16    | 8    | cipher type. 0 no cipher |
| 24    | 8    | compression type. 0 no compression. |
| 32    | 8    | signature position (0 == no signature header) |
| 40    | 8    | index position (0 == no index header) |
| 48    | 8    | unused1 (0) |
| 56    | 8    | unused2 (0) |

Item Header
-----------

Every item in a NAR archive need to start by an item header. The header is 256bits (4 Word64, 32 bytes) and is composed of:

| Index | Size | Description |
| ------| ---- | ----------- |
| 0     | 8    | item header signature (usually ASCII string) |
| 8     | 8    | flags    |
| 16    | 8    | length 1 |
| 24    | 8    | length 2 |

The header is directly followed by the item payload. The next item header can be found at offset:

    current item header position + 32 bytes + ROUND_UP64(length1) + ROUND_UP64(length2)

