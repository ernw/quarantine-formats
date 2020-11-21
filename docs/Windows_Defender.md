# Windows Defender

## Location of quarantine files

`/ProgramData/Microsoft/Windows Defender/Quarantine`

## Files and file name structure

There are three subdirectories in the Windows Defender quarantine directory:

- Entries
- ResourceData
- Resources

### Entries directory

The file names in this directory consist of some kind of GUID enclosed by curly brackets:

`\{[0-9A-F]{8}-[0-9A-F]{4}-[0-9A-F]{4}-[0-9A-F]{4}-[0-9A-F]{12}\}`

The first and second quadruple seem to be always `0000`.

### ResourceData directory

File names in this directory seem to be SHA1 hashes. It is unknown how they are generated.

`[0-9A-F]{40}`

Every file is located in a subdirectory. The name of the subdirectory consists of two characters that are the first two characters of the file name:

`[0-9A-F]{2}`

### Resources directory

Files in this directory follow the same schema as the files in the `ResourceData` directory.

For every file in this directory there is a matching file in the `ResourceData` directory with the same name.

## Structure of quarantine files

Files in the different directories serve different purposes:

- Entries: Timestamps and locations of quarantined files.
- ResourceData: The actual quarantine files containing the malicious files.
- Resources: Some metadata about quarantined files.


### Entries files

An Entries file consists of three parts:

- header
- data1
- data2

Each of the parts is encrypted individually using RC4 (see below for key). After decryption, it is possible to analyze the file.

*Note: In the following tables, offsets are relative to section starts.*

#### header section


| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Magic bytes: 0xdb, 0xe8, 0xc5, 0x01, 0x01, 0, 0x01, 0, 0, 0, 0, 0, 0, 0, 0, 0                       |
| 0x10    | Unknow data (0x18 bytes)                                                                            |
| 0x28    | Length of data1 section in bytes (uint4)                                                            |
| 0x2C    | Length of data2 section in bytes (uint4)                                                            |
| 0x30    | Unknown data (0x0C bytes)                                                                           |

Length of section: fixed, 0x3C.

#### data1 section

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | GUID, 0x10 bytes                                                                                    |
| 0x10    | Unknown data (0x10 bytes)                                                                           |
| 0x20    | Timestamp (0x08 bytes)                                                                              |
| 0x28    | Part of GUID from above, only first three parts (0x08 bytes)                                        |
| 0x30    | Number of strings (uint4)                                                                           |
| 0x34    | Malware type, null-terminated UTF8 string (dynamic length)                                          |

Length of section: dynamic, at least 0x34.

#### data2 section

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Number of entries in list (uint4)                                                                   |
| 0x04    | List of offsets where entries start (uint4, repeated 'number of entries' times)                     |
| ?       | List of entries (entry data structure is repeated 'number of entries' times)                        |


Entry data structure:

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Path, null-terminated UTF16LE                                                                       |
| ?       | Number of metadata elements stored in this entry                                                    |
| ?       | Type, null-terminated UTF8                                                                          |
| ?       | Padding (aligns following data at 4-bytes steps)                                                    |
| ?       | Metadata list elements (repeated 'number of metadata elements' times)                               |


Metadata elements data structure:

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Length of data (uint2)                                                                              |
| 0x02    | Subtype of data (uint1)                                                                             |
| 0x03    | Type of data (uint1)                                                                                |
| 0x04    | Data                                                                                                |
| ?       | Padding (aligns following data at 4-bytes steps)                                                    |

The data can be the following:

| Type of data | Description | Length  |
|--------------|-------------|---------|
| 0x20         | UTF16LE     | dynamic |
| 0x30         | uint4       | 0x04    |
| 0x40         | GUID        | 0x10    |
| 0x50         | uint8       | 0x08    |
| 0x60         | timestamp   | 0x08    |


### ResourceData files

ResourceData files are encrypted using RC4 (see below for key). After decryption they can be analyzed:

| Offset         | Description                                                                                         |
|----------------|-----------------------------------------------------------------------------------------------------|
| 0x00           | Binary security descriptor (see Binary_Security_Descriptor.md)                                      |
| bsd + 0x00     | Unknown (8 bytes)                                                                                   |
| bsd + 0x08     | Length of malicious file in bytes (uint8)                                                           |
| bsd + 0x10     | Unknown (4 bytes)                                                                                   |
| bsd + 0x14     | The malicious file                                                                                  |

After the malicious file, there might be some trailing 0-bytes.

### Resources files

A Resources file consists of three parts:

- header
- data1
- data2

Each of the parts is encrypted individually using RC4 (see below for key). After decryption, it is possible to analyze the file.

*Note: In the following tables, offsets are relative to section starts.*

#### header section


| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Magic bytes: 0xdb, 0xe8, 0xc5, 0x01, 0x01, 0, 0x01, 0, 0, 0, 0, 0, 0, 0, 0, 0                       |
| 0x10    | Unknow data (0x18 bytes)                                                                            |
| 0x28    | Length of data1 section in bytes (uint4)                                                            |
| 0x2C    | Length of data2 section in bytes (uint4)                                                            |
| 0x30    | Unknown data (0x0C bytes)                                                                           |

Length of section: fixed, 0x3C.

#### data1 section

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Unknown data (0x08 bytes)                                                                           |
| 0x10    | File ID (0x14 bytes)                                                                                |
| 0x20    | Unknown data (0x04 bytes)                                                                           |

Length of section: fixed, 0x20 bytes.

#### data2 section

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | GUID (0x10 bytes), this GUID references the name of an Entries file                                 |

Length of section: fixed, 0x10 bytes.



## Steps to analyze

The following steps are executed to gather the data from the different files:

- Parse the `Resources` file and get malicious file.
- From the name of the `Resources` files, derive the `ResourceData` file that belongs to this quarantine file.
- Parse the `ResourceData` file and get the contained GUID.
- From the GUID, derive the name of the `Entries` file that belongs to this `ResourceData` file.
- Parse the `Entries` file.
- From the `Entries` file, gather the location and timestamp of the original malicious file.



## RC4 decryption key

```
0x1E, 0x87, 0x78, 0x1B, 0x8D, 0xBA, 0xA8, 0x44, 0xCE, 0x69,
0x70, 0x2C, 0x0C, 0x78, 0xB7, 0x86, 0xA3, 0xF6, 0x23, 0xB7,
0x38, 0xF5, 0xED, 0xF9, 0xAF, 0x83, 0x53, 0x0F, 0xB3, 0xFC,
0x54, 0xFA, 0xA2, 0x1E, 0xB9, 0xCF, 0x13, 0x31, 0xFD, 0x0F,
0x0D, 0xA9, 0x54, 0xF6, 0x87, 0xCB, 0x9E, 0x18, 0x27, 0x96,
0x97, 0x90, 0x0E, 0x53, 0xFB, 0x31, 0x7C, 0x9C, 0xBC, 0xE4,
0x8E, 0x23, 0xD0, 0x53, 0x71, 0xEC, 0xC1, 0x59, 0x51, 0xB8,
0xF3, 0x64, 0x9D, 0x7C, 0xA3, 0x3E, 0xD6, 0x8D, 0xC9, 0x04,
0x7E, 0x82, 0xC9, 0xBA, 0xAD, 0x97, 0x99, 0xD0, 0xD4, 0x58,
0xCB, 0x84, 0x7C, 0xA9, 0xFF, 0xBE, 0x3C, 0x8A, 0x77, 0x52,
0x33, 0x55, 0x7D, 0xDE, 0x13, 0xA8, 0xB1, 0x40, 0x87, 0xCC,
0x1B, 0xC8, 0xF1, 0x0F, 0x6E, 0xCD, 0xD0, 0x83, 0xA9, 0x59,
0xCF, 0xF8, 0x4A, 0x9D, 0x1D, 0x50, 0x75, 0x5E, 0x3E, 0x19,
0x18, 0x18, 0xAF, 0x23, 0xE2, 0x29, 0x35, 0x58, 0x76, 0x6D,
0x2C, 0x07, 0xE2, 0x57, 0x12, 0xB2, 0xCA, 0x0B, 0x53, 0x5E,
0xD8, 0xF6, 0xC5, 0x6C, 0xE7, 0x3D, 0x24, 0xBD, 0xD0, 0x29,
0x17, 0x71, 0x86, 0x1A, 0x54, 0xB4, 0xC2, 0x85, 0xA9, 0xA3,
0xDB, 0x7A, 0xCA, 0x6D, 0x22, 0x4A, 0xEA, 0xCD, 0x62, 0x1D,
0xB9, 0xF2, 0xA2, 0x2E, 0xD1, 0xE9, 0xE1, 0x1D, 0x75, 0xBE,
0xD7, 0xDC, 0x0E, 0xCB, 0x0A, 0x8E, 0x68, 0xA2, 0xFF, 0x12,
0x63, 0x40, 0x8D, 0xC8, 0x08, 0xDF, 0xFD, 0x16, 0x4B, 0x11,
0x67, 0x74, 0xCD, 0x0B, 0x9B, 0x8D, 0x05, 0x41, 0x1E, 0xD6,
0x26, 0x2E, 0x42, 0x9B, 0xA4, 0x95, 0x67, 0x6B, 0x83, 0x98,
0xDB, 0x2F, 0x35, 0xD3, 0xC1, 0xB9, 0xCE, 0xD5, 0x26, 0x36,
0xF2, 0x76, 0x5E, 0x1A, 0x95, 0xCB, 0x7C, 0xA4, 0xC3, 0xDD,
0xAB, 0xDD, 0xBF, 0xF3, 0x82, 0x53
```


## Additional Useful Information: Scan History

Windows Defender stores a history of scans (one file per scan) in:

`/ProgramData/Microsoft/Windows Defender/Scans/History/Service/DetectionHistory/<number>/`

The scan history files are stored in a binary format and are basically a list of values. Each value consists of:

- Length of stored data
- Type of stored data: uint4
  - Types 0x00, 0x05, 0x06: uint4
  - Type 0x08: uint8
  - Type 0x0a: Windows timestamp (but maybe also something else)
  - Type 0x15: UTF-16LE string, null-terminated
  - Type 0x28: Threat Tracking information (see below)
  - Type 0xe1: raw data
- Content
- Padding, alignment on 8 bytes.

The Threat Tracking information consists of a header and a list of entries (key-value pairs):

- Header:
  - Length of data: uint4
- Entry:
  - Length of data: uint4
  - Name: UTF-16LE, null-terminated
  - Type of data: uint4
    - Type 0x03: flags, uint4
    - Type 0x04: Windows timestamp (but maybe also something else)
    - Type 0x05: boolean, uint1 (probably 0x0 or 0x1)
    - Type 0x06: UTF-16LE with encoded length (the length is a uint4 in front of the string)
  - Content


The scan history files store the SHA256 of the detected malware. By this a mapping to the quarantine files is possible.

The order of the different fields in the file seems to follow always the same order. The first UTF-16LE string that does not start with "Magic.Version" contains the malware name:

| Field    | Description                                                                                         |
|----------|-----------------------------------------------------------------------------------------------------|
| $name    | Malware name                                                                                        |
| $name+14 | File                                                                                                |
| $name+17 | Threat Tracking information                                                                         |
| $name+18 | Timestamp1                                                                                          |
| $name+26 | Name of exe                                                                                         |
| $name+30 | Timestamp2                                                                                          |
| $name+30 | Timestamp2                                                                                          |
| $name+36 | User                                                                                                |


The Threat Tracking information store three interesting fields:

- ThreatTrackingStartTime
- ThreatTrackingSha256
- ThreatTrackingSha1

## Additional Resources

See this helpful blogpost about the Windows Defender quarantine fomat:

- http://jon.glass/blog/quarantines-junk/


