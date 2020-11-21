# Symantec Endpoint Protection

## Location of quarantine files

The directory `Quarantine` contains metadata files (`<meta>.VBN`) and subdirectories which contain the files holding the actual quarantined files (`<meta>/<quarantine>.VBN`)

TODO: Full path to Quarantine directory.

## File name structure

`[0-9A-F]{8}.VBN` for metadata files and quarantine files.

## Structure of quarantine files

The file consists of a header (containing metadata about the quarantined file) and an XORed payload. The payload is a list of data structures and contains i.a. the XORed malicious file.
The payload is XORed with the key `0x5A`. After decrypting the payload, the quarantined file content can be decrypted with the key `0xFF`.

### Header

| Offset | Descryption                                        |
|--------|----------------------------------------------------|
| 0x00   | Offset of payload (usually 0x1290)                 |
| 0x04   | File name of quarantined file (UTF-8)              |
| 0x184  | CSV list of metadata (UTF-8)                       |
| 0x984  | Unknown (0x04 bytes)                               |
| 0x988  | Unix timestamp, more than a year in future         |
| 0x98C  | Winfiletime                                        |
| 0x994  | Winfiletime                                        |
| 0x99C  | Winfiletime                                        |
| 0x9a4  | Unknown (0x04 bytes)                               |
| 0xb8C  | "FileSystem"                                       |
| 0xbbC  | Unknown (0x04 bytes)                               |
| 0xbc0  | File name of Symantec tmp file (UTF-8)             |
| 0xd70  | Unix timestamp, maybe quarantine time              |

TODO: Find out the meaning of more header fields.

### Payload

Offsets are relative to the start of the payload.

| Offset  | Descryption                                       |
|---------|---------------------------------------------------|
| 0x00    | 8 zero bytes                                      |
| 0x08    | u8, offset of start of the metadata section ($om) |
| 0x10    | u8, length of the metadata section                |
| 0x18    | u8, offset of start of the content section  ($oc) |
| 0x20    | u8, length of the content section                 |
| $om     | metadata section, list of values                  |
| $oc     | content section, list of values                   |


A value consists of a single-byte ID indicating the type of encoded data. Some values have a fixed-length content, some of them have a dynamic-length content. If the content length is dynamic, the length is encoded as an u4 after the ID.

| ID      | fixed length | Description                                                                |
|---------|--------------|----------------------------------------------------------------------------|
| 0x01    | yes, u1      | Unknown                                                                    |
| 0x03    | yes, u4      | Unknown                                                                    |
| 0x04    | yes, u8      | Total size (in bytes) of quarantined file                                  |
| 0x06    | yes, u4      | Unknown                                                                    |
| 0x08    | no           | UTF-16LE encoded, null-terminated string                                   |
| 0x09    | no           | Raw data stream, may contain a list of values or XORed file content (0xFF) |
| 0x0a    | yes, u1      | Unknown                                                                    |


#### Metadata section

The metadata section seems to consist of several lists. Each list has a header and entries. Headers and entries consist of several values (see description above).

A list header consists of 5 values in this order:

- type 0x06: content seems to be always 1
- type 0x0A: content seems to be always 1
- type 0x0A: content seems to be always 0
- type 0x06: content seems to be always 1
- type 0x03: content is the number of entries in the following list.

Each list entry consists of 3 or 4 values:

- type 0x03: content is an ID in the list. IDs are ordered ascending, but are not necessarily consecutive, e.g. 0, 1, 45, 47.
- type 0x01: content seems to indicate the type of data that is encoded in this list entry.
  - content 0x11: Two type 0x09 will follow. The first will be a checksum, the second a raw data stream. The checksum is computed with the content of the raw stream.
  - content 0x07: One type 0x03 will follow.
  - content 0x01: One type 0x0A will follow.
  - content 0x09: One type 0x04 will follow.

#### Content section

The content section does not contain a list like the metadata section.

There is a type 0x08 field containing the SHA1 hash of the quarantined file.

There is a type 0x09 data stream containing the length of the quarantined file in bytes.

There is another type 0x08 field containing the security descriptor of the quarantined file.

The quarantined file's content may be split into several chunks. Each chunk is then stored in its own 0x09 raw data stream. The max chunk size seems to be 0x10000.

To find the quarantined file's content, one have to search in the content section for the 0x04 entry, indicating the file length. All the following 0x09 raw data streams can be XORed with 0xFF and combined to the original malicious file.


## Structure of metadata files

The metadata files are quite similar to the quarantine files. The differences are:

- They contain slightly different (but often redundant) information.
- They do not contain the quarantined file.
- They contain the malware name.
- The payload is not encrypted.

### Header

| Offset | Descryption                                        |
|--------|----------------------------------------------------|
| 0x00   | Offset of payload (usually 0x1290)                 |
| 0x04   | File name of quarantined file (UTF-8)              |
| 0x184  | CSV list of metadata (UTF-8)                       |

TODO: Find out the meaning of more header fields.


### Payload

The payload section is not encrypted. It basically is structured the same way as the metadata section in the quarantine files (list structure).