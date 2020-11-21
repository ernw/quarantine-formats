# Binary Security Descriptor

Some AV software stores binary security descriptors.

Documentation at microsoft.com:

- https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/2918391b-75b9-4eeb-83f0-7fdc04a5c6c9
- https://docs.microsoft.com/en-us/windows/win32/secauthz/sid-strings
- https://docs.microsoft.com/en-us/windows/win32/secauthz/well-known-sids

Other documentation:

- https://itconnect.uw.edu/wares/msinf/other-help/understanding-sddl-syntax/

## Header


| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Magic bytes: 0x03, 0x00, 0x00, 0x00, 0x02, 0x00, 0x00, 0x00                                         |
| 0x08    | Length of binary SD (uint4)                                                                         |
| 0x0C    | Padding (8 bytes)                                                                                   |
| 0x14    | Binary SD                                                                                           |


## Binary SD

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Revision (uint1)                                                                                    |
| 0x01    | Reserved (1 byte)                                                                                   |
| 0x02    | Control flags (uint2)                                                                               |
| 0x04    | Owner offset (uint4), points to SID                                                                 |
| 0x08    | Group offset (uint4), points to SID                                                                 |
| 0x0C    | SACL offset (uint4), points to ACL                                                                  |
| 0x10    | DACL offset (uint4), points to ACL                                                                  |

*Note*: The offsets can be 0. For example, if the SACL offset is 0, then there is no SACL encoded.

## SID

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Revision (uint1)                                                                                    |
| 0x01    | Number of uint4s encoded at the end of the binary SID                                               |
| 0x02    | Reserved (2 bytes)                                                                                  |
| 0x04    | First part of SID after 1- (uint4, big endian)                                                      |
| 0x08    | uint4, repeated 'number of uint4s' times (can also be 0 times)                                      |

## ACL

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Revision (uint1)                                                                                    |
| 0x01    | Reserved (1 byte)                                                                                   |
| 0x02    | Size of ACL in bytes (uint2)                                                                        |
| 0x04    | Number of ACEs (uint2)                                                                              |
| 0x06    | Reserved (2 bytes)                                                                                  |
| 0x08    | Encoded ACEs ('number of ACEs' times, can also be 0 times)                                          |


## ACE

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Access types (uint1)                                                                                |
| 0x01    | Flags (uint1)                                                                                       |
| 0x02    | Size of ACE (uint2)                                                                                 |
| 0x04    | Access mask (uint4)                                                                                 |
| 0x08    | SID                                                                                                 |
