# G Data Antivirus

## Location of quarantine files

`/ProgramData/G Data/AVK/Quarantine/`


## File name structure

Quarantine files seem to contain the host name of the computer, a timestamp and some kind of ID:

`<hostname> DDMMYYHHmmss[0-9A-F]{8}.q`

## Structure of quarantine files

Quarantine files consist of three parts:

- Data1: Contains metadata about the detected malware.
- Data2: Contains metadata about the location, timestamps and size.
- Data3: Contains the actual malicious file.

Each of the sections is encrypted individually, but data1 and data2 each have unencrypted magic bytes and length fields.

### Data1:

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | [0xca, 0xfe, 0xba, 0xbe]                                                                            |
| 0x04    | Length of the data in bytes (uint4)                                                                 |
|         | Encryption starts  here                                                                             |
| 0x08    | Unknown data (0x0C bytes)                                                                           |
| 0x14    | Unix timestamp of detection/quarantine creation                                                     |
| 0x18    | Unknown data (0x04 bytes)                                                                           |
| 0x1C    | Name of malware detected (UTF-16, with BOM and string length encoded)                               |

### Data2:

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | [0xba, 0xad, 0xf0, 0x0d]                                                                            |
| 0x04    | Length of the data in bytes (uint4)                                                                 |
|         | Encryption starts  here                                                                             |
| 0x08    | Unknown data (0x08 bytes)                                                                           |
| 0x10    | File size of quarantined file in bytes (uint4)                                                      |
| 0x14    | String (emtpy in samples, UTF-16, with BOM and string length encoded)                               |
| ?       | Unknown data (0x08 bytes)                                                                           |
| ?       | Windows timestamp (uint8)                                                                           |
| ?       | Windows timestamp (uint8)                                                                           |
| ?       | Windows timestamp (uint8)                                                                           |
| ?       | Unknown data (0x04 bytes)                                                                           |
| ?       | File size of quarantined file in bytes (uint4)                                                      |
| ?       | Original path of malware detected (UTF-16, with BOM and string length encoded)                      |


### Data3:

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Malicious file (length: read until EOF)                                                             |


## Key for decryption

The key is hard-coded in the file `AVKQt.dll`, which is part of G Data Antivirus.

The key is located in the `.rdata` section.


```
    process: util.custom_arc4.custom_arc4([0xA7, 0xBF, 0x73, 0xA0, 0x9F, 0x03, 0xD3, 0x11,
                                           0x85, 0x6F, 0x00, 0x80, 0xAD, 0xA9, 0x6E, 0x9B])
```

Within the DLL file, the key is located at offset `0x8adb8` with a length of 16 bytes.

File version of the DLL used to identify the key (pedump output):

```
# StringTable 040904E4:
  CompanyName         :  "G DATA Software AG"
  FileDescription     :  "G DATA Security Software Quarantine Module"
  FileVersion         :  "1.3.19336.1288"
  RepoDate            :  "2019-09-25 14:11:15 +0200"
  RepoName            :  "b2c/gecko_hotfix1"
  RepoRevision        :  "c4b5901d773f5f02dacdb5d8326084a1f52d6eb8"
  RepoVersion         :  "1.0.0.0-20191202-202655"
  InternalName        :  "AVKQt"
  LegalCopyright      :  "© G DATA Software AG. All rights reserved."
  OriginalFilename    :  "AVKQt.dll"
  ProductName         :  "G DATA Security Software"
  ProductVersion      :  "1.3.0.0"

  VarFileInfo         :  [ 0x409, 0x4e4 ]
```