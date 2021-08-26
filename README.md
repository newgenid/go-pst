<h1 align="center">
  <br>
  <a href="https://github.com/mooijtech/go-pst"><img src="https://i.imgur.com/LIicreP.png" alt="go-pst" width="280"></a>
  <br>
  go-pst
  <br>
</h1>

<h4 align="center">A library for reading PST files (written in Go/Golang).</h4>

<p align="center">
  <a href="https://github.com/mooijtech/go-pst/blob/master/LICENSE.txt">
      <img src="https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square">
  </a>
  <a href="https://github.com/mooijtech/go-pst/issues">
    <img src="https://img.shields.io/github/issues/mooijtech/go-pst.svg?style=flat-square">
  </a>
  <a href="https://github.com/mooijtech/go-pst">
      <img src="https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat-square">
  </a>
</p>

---

## Introduction

**go-pst** is a library for reading PST files (written in Go/Golang).

The PFF (Personal Folder File) and OFF (Offline Folder File) format is used to store Microsoft Outlook e-mails, appointments and contacts. The PST (Personal Storage Table), OST (Offline Storage Table) and PAB (Personal Address Book) file format consist of the PFF format.

## References

### Documentation

- [Personal Folder File (PFF) file format specification](https://github.com/mooijtech/go-pst/blob/master/docs/PFF.pdf)
- [Outlook Personal Folders (.pst) File Format](https://github.com/mooijtech/go-pst/blob/master/docs/MS-PST.pdf)

### Libraries

- [java-libpst](https://github.com/rjohnsondev/java-libpst)
- [libpff](https://github.com/libyal/libpff)
- [XstReader](https://github.com/Dijji/XstReader)
- [pstreader](https://github.com/Jmcleodfoss/pstreader)
- [PANhunt](https://github.com/Dionach/PANhunt/blob/master/pst.py)

## Datasets

This library is tested on the following datasets:

- [enron.pst](https://github.com/mooijtech/go-pst/blob/master/data/enron.pst)
- [32-bit.pst](https://github.com/mooijtech/go-pst/blob/master/data/32-bit.pst)

## Implementation

This implementation is based on the [references](#references).<br/>
The source code of go-pst will reference this implementation.

### File Header

| Offset        | Size          | Value                         | Description   |
| ------------- | ------------- | -------------                 | ------------- |
| 0             | 4             | "\x21\x42\x44\x4e" (**!BDN**) | The signature (magic identifier). |
| 8             | 2             |                               | The content type (client signature). See [Content Types](#content-types). |
| 10            | 2             |                               | The data version (NDB version). NDB is short for node database. See [Format Types](#format-types). |

### Content Types

| Value               | Description        |
| -------------       | -------------      |
| "\x53\x4d" (**SM**) | Used for PST files |
| "\x53\x4d" (**SO**) | Used for OST files |
| "\x41\x42" (**AB**) | Used for PAB files |

### Format Types

| Value               | Description        |
| -------------       | -------------      |
| 14                  | 32-bit ANSI format |
| 15                  | 32-bit ANSI format |
| 21                  | 64-bit Unicode format (by Visual Recovery) |
| 23                  | 64-bit Unicode format |
| 36                  | 64-bit Unicode format with 4k |

### The 64-bit header data

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 224           | 8             | The node b-tree file offset. |
| 240           | 8             | The block b-tree file offset. |
| 513           | 1             | Encryption type. See [Encryption Types](#encryption-types). |

### The 32-bit header data

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 188           | 4             | The node b-tree file offset. |
| 196           | 4             | The block b-tree file offset. |
| 461           | 1             | Encryption type. See [Encryption Types](#encryption-types). |

### Encryption Types

| Value        | Identifier          | Description   |
| -------------| -------------       | ------------- |
| 0x00         | NDB_CRYPT_NONE      | No encryption. |
| 0x01         | NDB_CRYPT_PERMUTE   | Compressible encryption. |
| 0x02         | NDB_CRYPT_CYCLIC    | High encryption. |

### The node and block b-tree

The following offsets start from the (node/block) b-tree offset.

#### 64-bit

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  488            | B-tree node entries (number of entries x entry size). |
| 488           |  1            | The number of entries. |
| 490           |  1            | The size of an entry. |
| 491           |  1            | B-tree node level. A zero value represents a leaf node. A value greater than zero represents a branch node, with the highest level representing the root. |

#### 64-bit 4k

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  4056            | B-tree node entries (number of entries x entry size). |
| 4056           |  2            | The number of entries. |
| 4060           |  1            | The size of an entry. |
| 4061           |  1            | B-tree node level. A zero value represents a leaf node. A value greater than zero represents a branch node, with the highest level representing the root. |

#### 32-bit

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  496            | B-tree node entries (number of entries x entry size). |
| 496           |  1            | The number of entries. |
| 498           |  1            | The size of an entry. |
| 499           |  1            | B-tree node level. A zero value represents a leaf node. A value greater than zero represents a branch node, with the highest level representing the root. |

### The b-tree entries

#### The 64-bit block b-tree branch node entry

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  8            | [The identifier](#identifier) of the first child node. 32-bit integer. |
| 16            |  8            | The file offset, points to a block b-tree branch or leaf node entry. |

#### The 64-bit block b-tree leaf node entry

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  8            | [The identifier](#identifier). 32-bit integer. |
| 8             |  8            | The file offset, points to a [Heap-on-Node](#heap-on-node). |
| 16            |  2            | The size of the [Heap-on-Node](#heap-on-node). |

#### The 64-bit node b-tree leaf node entry

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  8            | [The identifier.](#identifier) 32-bit integer. |
| 8             |  8            | The node identifier of the data. This node identifier is found in the block b-tree. |
| 16            |  8            | The node identifier of the local descriptors. This node identifier is found in the block b-tree. |

#### The 32-bit block b-tree branch node entry

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  4            | [The identifier](#identifier) of the first child node. 32-bit integer. |
| 8             |  4            | The file offset, points to a block b-tree branch or leaf node entry. |

#### The 32-bit block b-tree leaf node entry

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  4            | [The identifier](#identifier). 32-bit integer. |
| 4             |  4            | The file offset, points to a [Heap-on-Node](#heap-on-node). |
| 8             |  2            | The size of the [Heap-on-Node](#heap-on-node). |

#### The 32-bit node b-tree leaf node entry

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  4            | [The identifier](#identifier). 32-bit integer. |
| 4             |  4            | The node identifier of the data. This node identifier is found in the block b-tree. |
| 8             |  4            | The node identifier of the local descriptors. This node identifier is found in the block b-tree. |

### Identifier

The 32-bit integer (identifier) can be used to search for b-tree nodes.

| Offset        | Size          | Description   |
| ------------- | ------------- | ------------- |
| 0             |  5 bits       | [The identifier type](#identifier-types). |

#### Identifier types

| Value         | Identifier          | Description   |
| ------------- | ------------- | ------------- |
| 0          |  HID       | Table value (or heap node) |
| 1          |  INTERNAL       | Internal node |
| 2          |  NORMAL_FOLDER       | Folder item |
| 3          |  SEARCH_FOLDER       | Search folder item |
| 4          |  NORMAL_MESSAGE       | Message item |
| 5          |  ATTACHMENT       | Attachment item |
| 6          |  SEARCH_UPDATE_QUEUE       | Queue of changed search folder items |
| 7          |  SEARCH_CRITERIA_OBJECT       | Search folder criteria |
| 8          |  ASSOCIATED_MESSAGE       | Associated contents item |
| 10          |  CONTENTS_TABLE_INDEX       | Unknown |
| 11          |  RECEIVE_FOLDER_TABLE       | Inbox item (or received folder table) |
| 12          |  OUTGOING_QUEUE_TABLE       | Outbox item (or outgoing queue table) |
| 13          |  HIERARCHY_TABLE       | Sub folders item (or hierarchy table) |
| 14          |  CONTENTS_TABLE       | Sub messages items (or contents table) |
| 15          |  ASSOCIATED_CONTENTS_TABLE       | Sub associated contents item (or associated contents table) |
| 16          |  SEARCH_CONTENTS_TABLE       | Search contents table |
| 17          |  ATTACHMENT_TABLE       | Attachments item |
| 18          |  RECIPIENT_TABLE       | Recipients items |
| 19          |  SEARCH_TABLE_INDEX       | Unknown |
| 20          |         | Unknown |
| 21          |         | Unknown |
| 22          |         | Unknown |
| 23          |         | Unknown |
| 24          |         | Unknown |
| 31          |  LTP       | Local descriptor value |
| 290          |  ROOT_FOLDER       | The root folder. |

### Heap-on-Node

If the encryption type was set in the file header, **the entire Heap-on-Node is encrypted**.

#### Compressible encryption

Compressible encryption is a simple [byte-substitution cipher](https://github.com/rjohnsondev/java-libpst/blob/develop/src/main/java/com/pff/PSTObject.java#L843) with a fixed [substitution table](https://github.com/rjohnsondev/java-libpst/blob/develop/src/main/java/com/pff/PSTObject.java#L725).

#### Heap-on-Node HID

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  5 bits       | HID Type; MUST be set to 0 (IdentifierTypeHID) to indicate a valid HID. |
| 0.5           |  11 bits      | HID index. The index value that identifies an item allocated in the [allocation table](#heap-on-node-page-map). |
| 2.0           |  16 bits      | This number indicates the index of the data block in which this heap item resides.  |

#### Heap-on-Node header

The first block contains the Heap-on-Node header.

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  2            | The offset to the [Heap-on-Node page map](#heap-on-node-page-map). |
| 2             |  1            | Block signature; MUST be set to 0xEC (236) to indicate a Heap-on-Node. |
| 3             |  1            | [The table type](#table-types). |
| 4             |  4            | [HID](#heap-on-node-hid) User Root. |

#### Heap-on-Node bitmap header

Blocks 8, 136, then every 128th contains the Heap-on-Node bitmap header (``i == 8 || i >= 138 && (i - 8) / 128 == 0``).

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  2            | The offset to the [Heap-on-Node page map](#heap-on-node-page-map) (starting at the Heap-on-Node page header). |


#### Heap-on-Node page header

This is only used when multiple Heap-on-Node blocks are present.

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  2            | The offset to the [Heap-on-Node page map](#heap-on-node-page-map) (starting at the Heap-on-Node page header). |

#### Heap-on-Node page map

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  2            | Allocation count. |
| 4             |  variable     | Allocation table. This contains Allocation count + 1 entries. Each entry is an int (16 bit) value that is the byte offset to the beginning of the allocation. The start of this offset can be retrieved by using ``page map offset + (2 * hidIndex) + 2`` (page map offset plus the start of the allocation table, at the HID index offset). An extra entry exists at the Allocation count +1 position to mark the offset of the next available slot. |

#### Table types

| Table type    | Description   | Features   |
| ------------- | ------------- | ------------- |
| 108             |  6c table            | Has a b5 table header. |
| 124             |  7c table            | Table Context. Has a b5 table header. |
| 140             |  8c table            | Has a b5 table header |
| 156             |  9c table            | Has a b5 table header. |
| 165             |  a5 table            |  |
| 172             |  ac table            | Has a b5 table header. |
| 181             |  b5 table header     | B-Tree on Heap |
| 188             |  bc table            | Property Context. Has a b5 table header. |
| 204             |  cc table            | Unknown |

### B-Tree-on-Heap

#### B-Tree-on-Heap header

All tables should have a BTree-on-Heap header at [HID](#heap-on-node-hid) **0x20** (the start offset to the BTree-on-Heap header is in the [allocation table](#heap-on-node-page-map)).
This is the HID User Root from the [Heap-on-Node header](#heap-on-node-header).

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  1            | Table type. MUST be 188. |
| 1             |  1            | Size of the BTree Key value. MUST be 2, 4, 8 or 16. |
| 2             |  1            | Size of the data value. MUST be greater than zero and less than or equal to 32. |
| 3             |  1            | Index depth. |
| 4             |  4            | [HID](#heap-on-node-hid) root. Points to the B-Tree-on-Heap entries (offset can be found in the [allocation table](#heap-on-node-page-map)). |

### Property Context

The property context starts at the [HID root](#b-tree-on-heap-header) of the B-Tree-on-Heap header.

#### Property Context B-Tree-on-Heap Record

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  2            | Property ID.  |
| 2             |  2            | [Property type](#property-types).  |
| 4             |  4            | Reference HNID (HID or NID). In the event where the Reference HNID contains an HID or NID, the actual data is stored in the corresponding heap or subnode entry, respectively.  |

#### Property types

| Type        | Value    | 
| ------------- | ------------- |
| Integer16 | 2 |
| Integer32 | 3 |
| Floating32 | 4 |
| Floating64 | 5 |
| Currency | 6 |
| FloatingTime | 7 |
| ErrorCode | 10 |
| Boolean | 11 |
| Integer64 | 20 |
| String | 31 |
| String8 | 30 |
| Time | 64 |
| GUID | 72 |
| ServerID | 251 |
| Restriction | 253 |
| RuleAction | 254 |
| Binary | 258 |
| MultipleInteger16 | 4098 |
| MultipleInteger32 | 4099 |
| MultipleFloating32 | 4100 |
| MultipleFloating64 | 4101 |
| MultipleCurrency | 4102 |
| MultipleFloatingTime | 4103 |
| MultipleInteger64 | 4116 |
| MultipleString | 4127 |
| MultipleString8 | 4126 |
| MultipleTime | 4160 |
| MultipleGUID | 4168 |
| MultipleBinary | 4354 |
| Unspecified | 0 |
| Null | 1 |
| Object | 13 |

### The root folder

The [identifier](#identifier-types) 290 refers to the root folder.

### Sub-Folders

A folder [identifier](#identifier-types) + 11 refers to the related sub folders (for example, for the root folder this is 290 + 11 = 301).

The related sub folders consists of the Table Context (7c table).

### Table Context

The table context has a B-Tree-on-Heap.

The table context starts at the [HID User Root](#heap-on-node-header) of the Heap-on-Node.

#### Table Context Info

| Offset        | Size          | Description   | 
| ------------- | ------------- | ------------- |
| 0             |  1            | [Table context signature](#table-types). MUST be 124.  |
| 1             |  1            | Column count (number of columns in the table context).  |
| 14            |  4            | HNID to the Row Matrix (the actual table data).  |
| 22            |  variable     | [Column Descriptors](#table-context-column-descriptor). |


#### Table Context Column Descriptor

## Contact

Feel free to contact me if you have any questions.<br/>
**Name**: Marten Mooij<br/>
**Email**: info@mooijtech.com<br/>
**Phone**: +31 6 30 53 47 67

## License

[MIT](https://github.com/mooijtech/go-pst/blob/master/LICENSE.txt)
