# NAR (New ARchiver)

Author: Nicolas Di Prima, Vincent Hanquez

**Table of content**

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Specifications](#specifications)
  - [NAR Header](#nar-header)
    - [Magic Signature](#magic-signature)
    - [Flags](#flags)
    - [Data](#data)
  - [Standard Items](#standard-items)
    - [Initiate](#initiate)
    - [File](#file)
    - [Signature](#signature)
- [NAR Index](#nar-index)

## Introduction

What do we solve:

* simple format for packing data with simple length specification for filepath and content.
* integrate compression, encryption, authenticated data and signature.

## Requirements

* indexable / mmap friendly
* no filepath size limitation (within reason !)
* Word64 for length (2 exbibytes maximum per item)
* 64 bits little endian friendly, no per byte access, every fields aligned to 64 bits.

# Specifications

NAR works with a simple data representation. Every **NAR Item** composed of
three elements:

1. The NAR Header;
2. Data 1 (length specified in the NAR Header);
3. Data 2 (length specified in the NAR Header).

## NAR Header

The NAR Header provides the **NAR Item metadata**: a magic number, some flags
and tow length (for **Data 1** & **Data 2**).

Size : **32 bytes** (**4 Word64**) - unused bits set to 0

| Index (bytes) | Size (bytes) | Description |
|:------------- |:------------ |:----------- |
| 0             | 8            | magic signature (identify the type of Item) |
| 8             | 8            | flags (see below) |
| 16            | 8            | length 1 (0) (unsigned 64 bits integer) |
| 24            | 8            | length 2 (0) (unsigned 64 bits integer) |

### Magic Signature

The signature is a unique identifier for the type of the given **NAR Item**.
The NAR specification provides some standard **NAR Items** but users are free
to create their own API (keeping in mind that they might be breaking
compatibility with other NAR libraries).

### Flags

The Item Header Flags is split into 2 fields: one for common purpose at the lowest bits (10 bits), one for per-item purpose at the highest bits.

| Index (bit) | Size (bits) | Description |
|:----------- |:----------- |:----------- |
| 0           | 1           | compression applied for Data 1 |
| 1           | 1           | compression applied for Data 2 |
| 2           | 8           | not used yet (set to **0**) |
| 10          | 54          | per-item meaning |

#### Compression

When compression bit is set, the data starts with a compression header of 8 bytes to define which type of compression is used for the rest of the data stream.

### Data

The **NAR Header** is followed by two entities **Data 1** and **Data 2**.
Their length is specified into the **NAR header**. Each of them may be followed
by unused bits in order to meet the 64bits alignment requirement.

Then the item data are:

| Field    | Index (bytes)                 | Size (bytes) |
|:-------- |:----------------------------- |:------------ |
| Data 1   | 0                             | length1 |
| padding  | length1                       | ROUND_UP64(length1) - length1 |
| Data 2   | ROUND_UP64(length1)           | length2 |
| padding  | ROUND_UP64(length1) + length2 | ROUND_UP64(length2) - length2 |

## Standard Items

The following Items are standard and must be implemented.

### Initiate

This header is can be found in the 1st position when parsing a file or
a stream. It can provide metadata about **NAR Item** to follow in the stream.

**This item is optional**. But it is recommended to set it for future
compatibility as it contains the version of the NAR format used.

* Magic Signature:
  * **hexadecimal**: 0x5d204852414e205b;
  * **decimal**: 6710442962902196315;
  * **string**: [ NARH ];
* flags: see below;
* length1: unused;
* length2: unused.

|  0  |  1  | 2-9 | 10-45 | 46-63 |
|:---:|:---:|:---:|:-----:|:-----:|
|  0  |  0  |  0  |   0   | NAR Format Version |

### File

A file item start by a item header where the item values are / represent:

* Magic Signature:
  * **hexadecimal**: 0x5d204852414e205b;
  * **decimal**: 6710442962902196315;
  * **string**: [ FILE ];
* flags: see below;
* length1: filepath length;
* length2: file length (after compression. i.e. size of the blob).

|  0  |  1  | 2-9 | 10-62 | 63 |
|:---:|:---:|:---:|:-----:|:-----:|
| c1  | c2  |  0  |   0   | executable |

### Signature

Signed content starts with `Signature Start` item header and finish with a `Signature End` item header.
The start event starts a new cryptographic hashing context, where all further bytes will be appended to his hashing context until the end event, which will contains the cryptographic signature of the hash generated.

The signature start item must contains the hash algorithm to use.
The signature end item must contains the type of signature algorithm, and the signature itself.

# NAR Index

NAR Items have specific positions, and it may be useful to generate a fast lookup index.

format to be defined
