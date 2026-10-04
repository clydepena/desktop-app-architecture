# Streams and formats

The building blocks for reading and writing data piece by piece, and for encoding it as bytes.

## Streams

A **stream** reads or writes sequentially, without holding the whole file in memory. Essential for big files,
network data and compressed data.

### Decorators

Streams wrap other streams, each adding one feature (the **Decorator** pattern):

```
file stream -> buffered -> decompressing -> hashing -> your reader
```

- **buffering:** fewer, larger reads from the underlying source
- **decompressing / compressing**
- **hashing:** compute a checksum while reading, to verify integrity at the end
- **decrypting / encrypting**
- **counting / limiting:** stop a reader from reading past an expected size (important for untrusted data)

Each layer is small and reusable: a decompressor doesn't care whether its bytes come from a file, an archive entry
or the network. Java's IO library is the textbook example of this design.

### Ranges and seeking

Reading "bytes 4,096 to 8,191" of a file or archive entry lets streaming formats (audio, video, chunked worlds,
large textures with several detail levels) read only what they need. Archives with block compression make ranges
cheap even for compressed entries ([Packages](03-packages.md)).

## Formats: the generic parts

### Text

- **UTF-8** everywhere, without guessing encodings.
- **Line endings:** read both `\n` and `\r\n`; write one consistently.
- Text formats (JSON, TOML, XML, YAML) are readable and merge well in version control; good for settings, project
  files and anything people edit or review.

### Binary

- **Fixed-size integers and floats**, length-prefixed strings and arrays.
- **Endianness:** the byte order of multi-byte values. Pick one for the file format (usually little-endian) and
  convert on platforms that differ, rather than writing memory as it happens to be.
- **Alignment and padding**, if data is meant to be memory-mapped and used in place.
- Binary formats are compact and fast; good for runtime formats, large data and network messages.

### Compression

| Kind | Examples | Good for |
|---|---|---|
| **fast** | LZ4, Snappy | data loaded during play: decompression must keep up with the disk |
| **balanced** | zstd, deflate | general use, with adjustable levels |
| **strong** | LZMA, high zstd levels, Brotli | downloads and installs: decompressed once |

### Checksums and hashes

- **Checksums** (CRC32, xxHash): fast, detect accidental corruption.
- **Cryptographic hashes** (SHA-256): detect deliberate changes; used for **content addressing** (a file named by
  the hash of its contents, so identical files are stored once and changes are detected by comparing hashes).

## What belongs where

IO provides **how** bytes are read, written, compressed and checked. Deciding **what** goes into a document file and
which version it is belongs to [Persistence](../../tool-framework/03-persistence.md); describing the fields belongs
to the [data description](../08-data-description.md).
