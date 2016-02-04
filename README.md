
# ReGrid

This is the official spec for the [ReGrid](https://github.com/internalfx/regrid) Nodejs library.

- Authors [**Bryan Morris**](https://github.com/internalfx), [**Brian Chavez**](https://github.com/bchavez)
- Advisors [**Daniel Mewes**](https://github.com/danielmewes)

ReGrid is a method of storing large files inside a RethinkDB database.

### Features

- **Reliable** - Files are distributed across the cluster, benefiting from RethinkDB's automatic failover.
- **Scalable** - Easily store large files in RethinkDB, distributed across the cluster.
- **Consistent** - Sha256 hashes are calculated when the file is written, and verified when read back out.

### Overview

When a file is written to ReGrid, a **files** record is written to a **files table**. Then the file is broken up into **chunks** which are written as separate records in a **chunks table**. Once all the chunks are written, the **files** record is updated to show that the file is `Complete`. The file is now ready for read operations.

# Document Structure

#### Files record

```json
{
  "id" : "<String>",
  "length" : "<Number>",
  "chunkSizeBytes" : "<Number>",
  "finishedAt" : "<Time>",
  "startedAt" : "<Time>",
  "deletedAt" : "<Time>",
  "sha256" : "<String>",
  "filename" : "<String>",
  "status" : "<String>",
  "metadata" : "<Object>"
}
```

| Key | Description |
|---|---|
| id | a unique ID for this document. |
| length | the length of this stored file, in bytes. |
| chunkSizeBytes | the size, in bytes, of each data chunk of this file. This value is configurable by file. The default is 255KB (1024 * 255). |
| finishedAt | the date and time this file finished writing to ReGrid. The value of this field MUST be the datetime when the upload completed, not the datetime when it was begun. |
| startedAt | the date and time this file started writing to ReGrid. The value of this field MUST be the datetime when the upload started, not the datetime when it was finished. |
| deletedAt | the date and time this files status was set to `Deleted`. The value of this field MUST be the datetime when file was marked `Deleted`. |
| sha256 | SHA256 checksum for this user file, computed from the file’s data, stored as a hex string (lowercase). |
| filename | the name of this stored file; this does not need to be unique. |
| status | Status may be "Complete" or "Incomplete" or "Deleted". |
| metadata | any additional application data the user wishes to store. |

#### Chunks record

```json
{
  "id": "<String>",
  "files_id": "<String>",
  "num": "<Number>",
  "data": "<Binary>"
}
```

| Key | Description |
|---|---|
| id | a unique ID for this document. |
| files_id | the id for this file (the id from the files table document). |
| num | the index number of this chunk, zero-based |
| data | a chunk of data from the user file |
