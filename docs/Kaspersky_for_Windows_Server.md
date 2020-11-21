# Kaspersky for Windows Server

## Location of quarantine files

`/ProgramData/Kaspersky Lab/Kaspersky Security for Windows Server/<version>/Qurantine`

## File name structure

The metadata is stored in a file called `quarantine.db`.

All quarantine files containing the malware samples consist of a UUID:

`\{[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}\}`

## Structure of quarantine files

(Tested with version 10.1.)

The quarantine files just consist of the xored malware. The key is:

```
[0xe2, 0x45, 0x48, 0xec, 0x69, 0x0e, 0x5c, 0xac]
```

The metadata database ``quarantine.db`` has the following structure:


```sql
CREATE TABLE objattrs (uuid text primary key,attrs int,rights blob,creation_time int,access_time int,write_time int)
CREATE TABLE objects (uuid text primary key,path text,name text,virname text,virtype int,level int,time int,size int,status int,sent int,user text)
```

## Timestamps

The database stores the quarantine timestamp (column `objects.time`) in the following manner:

```
Timestamp conversion: 568581708202780672 = 2020-02-13 23:54:40 CET
t: 0000011111100100000000100000110100010111001101100010100000000000
Y: 1111111111111111000000000000000000000000000000000000000000000000
M: 0000000000000000111111110000000000000000000000000000000000000000
D: 0000000000000000000000001111111100000000000000000000000000000000
H: 0000000000000000000000000000000011111111000000000000000000000000
m: 0000000000000000000000000000000000000000111111110000000000000000
s: 0000000000000000000000000000000000000000000000001111111100000000
u: 0000000000000000000000000000000000000000000000000000000011111111
Bitmasks:
Y: FFFF000000000000
M: FF0000000000
D: FF00000000
H: FF000000
m: FF0000
s: FF00
u: FF
```

`u` is *u*nused and always zero.

**Note:** The timestamps are created in *local time*. This means, you need to know the timezone that was configured for this system, so that you can interprete the timestamp correctly.


## Security Descriptor

The column `objattrs.rights` stores a binary security descriptor. Documentation about the format can be found here:

- Binary_Security_Descriptor.md
