# Sophos Antivirus

## Location of quarantine files

`/ProgramData/Sophos Anti-Virus/Safestore/`


## Files and file name structure

There several file that need to be considered:
- `SafeStore.db` or `Safestore.db`: An encrypted SQLite database.
- `safestore_key.txt` or `SafeStore.pw`: A text file containing a UUID that is used as the password for decrypting the Safestore.db.
- `[0-9A-F]{40}.dat`: The actual quarinted files, but also metadata files for the quarantined files. The name seems to be a SHA1. Per quarantined file there seem to be two files: one contains the encrypted malicious file, one contains the metadata.

Note: There is also a file called `Quarantine.xml` which seems to store information about files. However, the files listed there could not be found on the file system.

## Structure of quarantine files

Database structure of the `Safestore.db`:

```sql
CREATE TABLE "BlobPropertyTable" (
	"blobpropertyid"	integer,
	"type"	varchar NOT NULL,
	"objectid"	integer NOT NULL,
	"blobid"	integer NOT NULL,
	FOREIGN KEY("blobid") REFERENCES "BlobTable"("blobid"),
	FOREIGN KEY("objectid") REFERENCES "ThreatObjectTable"("objectid"),
	PRIMARY KEY("blobpropertyid")
);

CREATE TABLE "BlobTable" (
	"blobid"	integer,
	"name"	varchar NOT NULL,
	"size"	integer NOT NULL,
	"originalchecksum"	varchar NOT NULL,
	"storedchecksum"	varchar NOT NULL,
	"blobversion"	varchar NOT NULL,
	PRIMARY KEY("blobid")
);

CREATE TABLE "MetaInfoTable" (
	"schema_vermajor"	integer NOT NULL,
	"schema_verminor"	integer NOT NULL,
	"schema_verpatch"	integer NOT NULL,
	"min_safestore_vermajor"	integer NOT NULL,
	"min_safestore_verminor"	integer NOT NULL,
	"min_safestore_verpatch"	integer NOT NULL,
	"file_encryption_salt"	varchar NOT NULL,
	"file_encryption_test"	varchar NOT NULL
);

CREATE TABLE "StringPropertyTable" (
	"stringpropertyid"	integer,
	"type"	varchar NOT NULL,
	"value"	varchar NOT NULL COLLATE nocase,
	"objectid"	integer NOT NULL,
	FOREIGN KEY("objectid") REFERENCES "ThreatObjectTable"("objectid"),
	PRIMARY KEY("stringpropertyid")
);

CREATE TABLE "ThreatObjectTable" (
	"objectid"	integer,
	"objectguid"	varchar NOT NULL,
	"threatguid"	varchar NOT NULL,
	"type"	varchar NOT NULL,
	"datetime"	integer NOT NULL,
	"safestoreversion"	varchar NOT NULL,
	"status"	varchar NOT NULL,
	PRIMARY KEY("objectid")
);

CREATE INDEX "BlobPropertyTable_Index" ON "BlobPropertyTable" (
	"objectid"	asc,
	"blobid"	asc
);

CREATE INDEX "BlobTable_Index" ON "BlobTable" (
	"originalchecksum"	asc
);

CREATE INDEX "StringPropertyTable_Index" ON "StringPropertyTable" (
	"objectid"	asc
);

CREATE INDEX "ThreatObjectTable_Index" ON "ThreatObjectTable" (
	"datetime"	asc
);
```
