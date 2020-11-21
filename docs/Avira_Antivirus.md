# Avira Antivirus

## Location of quarantine files

`/ProgramData/Avira/Antivirus/INFECTED`

## File name structure

`[0-9a-f]{8}.qua`

## Structure of quarantine files

The files consist of a header (containing metadata about the quarantined file) and the quarantined file itself (XORed with 0xAA).

| Offset  | Description                                                                                         |
|---------|-----------------------------------------------------------------------------------------------------|
| 0x00    | Magic bytes "AntiVir Qua" + five 0 bytes                                                            |
| 0x10    | Start byte of the XORed malicious file (uint4)                                                      |
| 0x14    | Length of file name in bytes (uint4)                                                                |
| 0x18    | Length of hostname in bytes (uint4), often empty                                                    |
| 0x3C    | Unix timestamp of the quarantine time                                                               |
| 0x9C    | String containing the malware type. 0-terminated UTF-8 string. Reserved space: 0x40                 |
| 0xDC    | String containing the path of the malicious file. UTF-16LE. Length determined before.               |
| 0xDC+?  | String containing the host of the malicious file. UTF-16LE. Length determined before.               |
| 0xDC+?  | Malicious file XORed with 0xAA. Length determined before. After reading this we should be at EOF.   |
