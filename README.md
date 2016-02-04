# NAR (New ARchiver)

Author: Nicolas DI PRIMA, Vincent HANQUEZ

## Introduction

What do we solve:

* simple format for packing data with simple length specification for filepath and content.
* integrate compression, encryption, authenticated data and signature.

## Requirements

* indexable / mmap friendly
* no filepath size limitation (within reason !)
* Word64 for length (2 exbibytes maximum per item)
* 64 bits little endian friendly, no per byte access, every fields aligned to 64 bits.


# Metadata

In this document:
* **Position** means the exact position in the NAR document (if it is in a File, it means it is the n-th byte).

## NAR Header

Size : 64 bytes (8 Word64) - unused bytes set to 0

| Index | Size | Description |
| ----- | ---- | ----------- |
| 0     | 8    | magic signature: ASCII string “[ NARH ]” |
| 8     | 8    | format version (upper 32 bits) + software version (lower 32 bits) |
| 16    | 8    | cipher type. 0 no cipher |
| 24    | 8    | compression type. 0 no compression. |
| 32    | 8    | signature **position** (optional, 0 == maybe no signature header)\* |
| 40    | 8    | index **position** (optional, 0 == maybe no index header)\* |
| 48    | 8    | unused1 (0) |
| 56    | 8    | unused2 (0) |

\* a server, streaming content to a client, might not be able to know the position of the signature or the index until the end of the stream. These **positions** are only given in order to facilitate access to these items.

## Item Header

Every item in a NAR archive need to start by an item header.

Every item in a NAR archive need to start by an item header. The header is 256bits (4 Word64, 32 bytes) and is composed of:

Size: 32 bytes (4 Word64) - unused bytes set to 0

| Index | Size | Description |
| ------| ---- | ----------- |
| 0     | 8    | item header signature (usually ASCII string) |
| 8     | 8    | flags    |
| 16    | 8    | length 1 |
| 24    | 8    | length 2 |

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


### Flags

The Item Header Flags is split into 2 fields: one for common purpose, one for per-item purpose.

| Index | Bits | Description |
| ----- | ---- | ----------- |
| 0     | 0    | compression content 1 |
|       | 1    | cipher content 1 |
|       | 2    | compression content 2 |
|       | 3    | cipher content 2 |
|       | 4-15 | not used yet. must be set to 0 for future purpose |
| 2     | 6    | saved for per-item purpose |
