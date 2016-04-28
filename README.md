# NAR (New ARchiver)

Author: Nicolas Di Prima, Vincent Hanquez

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

Size : 32 bytes (4 Word64) - unused bytes set to 0

| Index | Size | Description |
| ----- | ---- | ----------- |
| 0     | 8    | magic signature: ASCII string “[ NARH ]” |
| 8     | 8    | version (highest 16 bits) + other flags (0) |
| 16    | 8    | length 1 (0) |
| 24    | 8    | length 2 (0) |

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
  * bit 63 : executable
  * bit 11-62 : not defined. must be 0
* length 1: filepath length
* length 2: file length (after compression. i.e. size of the blob)

Then the item data are:

| Field      | Index in bytes                | Size in bytes                 |
| ---------- | ----------------------------- | ----------------------------- |
| filepath   | 0                             | length1                       |
| padding    | length1                       | ROUND_UP64(length1) - length1 |
| file data  | ROUND_UP64(length1)           | length2                       |
| padding    | ROUND_UP64(length1) + length2 | ROUND_UP64(length2) - length2 |


### Flags

The Item Header Flags is split into 2 fields: one for common purpose at the lowest bits (10 bits), one for per-item purpose at the highest bits.

| Bits | Description |
| ---- | ----------- |
| 0    | compression content 1 |
| 1    | compression content 2 |
| 2-9  | not used yet. must be set to 0 for future purpose |
| 10-63| per-item meaning |
