Author: Garret Rieger  
Date: January 14th, 2021  
Document Version: 2  

# Changes from Version 1

* Changed binary protocol to use a custom encoding which replaces protobuf.

# Overview

This document describes a protocol which  allows a client to retrieve a
subset of a font and then augment it by expanding the subset with
subsequent requests. Requests are sent to a server via HTTP POST requests.
Request information is encoded into the body of the POST request using a
custom binary encoding.

The following sections first describe the encoding for the request and
response payload. Then subsequent sections discuss the expected behaviour
of the client and server.

# Basic Data Types

| Data Type       | Description                                           |
| --------------- | ----------------------------------------------------- |
| VarBitSet       | Variable length bit set.                              |
| UInt64          | Unsigned 64 bit integer. Stored big-endian.           | 
| UIntBase128     | Variable-length encoding of 32-bit unsigned integers. |
| IntBase128      | Variable-length encoding of 32-bit signed integers.   |
| ArrayOf\<Type\> | Length delimited array of another `Type`.             |

## VarBitSet

A variable length bit set. Occupies one or more bytes. The length of the
bit set is determined by reading bytes until encountering one where the
most significant bit set to zero. For each byte the remaining 7 bits are
used as a bit set.

For example the bytes:
b10000100, b01100000

Encodes the bitset:

00001001100000

## UInt64

unsigned 64 bit integer. Exactly 4 bytes.

## UIntBase128

Variable length unsigned int, see WOFF2 specfication.´

## IntBase128

Variable length signed int, uses zig zag encoding.

TODO(garretrieger): more details.

## ArrayOf\<Type\>

Length delimited list of `Type`

```
struct {
  UIntBase128 length;
  Type values[length];
};
```

# Objects

An object is encoded as a series of optionally present fields. Each
field has an id number associated with it. Each fields value can
be any of the previously defined primitive data types, or another object.

An object is encoded as a VarBitSet which encodes which fields are present.
This is followed by the byte encoding of each field concatenated in order
of field id.

```
VarBitSet present_fields;
// If n bits are set to 1 in present_fields:
byte field_0[];
...
byte field_n[];
```

In present_fields if bit M is set to 1 that implies that data for the
field with ID = M is present.

## Request

  | ID | Field Name             | Type                 |
  | -- | ---------------------- | -------------------- |
  | 0  | protocol_version       | UIntBase128          |
  | 1  | original_font_checksum | Uint64               |
  | 2  | base_checksum          | Uint64               |
  | 3  | patch_format           | ArrayOf<UIntBase128> |
  | 4  | codepoints_have        | CompressedSet        |
  | 5  | codepoints_needed      | CompressedSet        |
  | 6  | index_checksum         | Uint64               |
  | 7  | indices_have           | CompressedSet        |
  | 8  | indices_needed         | CompressedSet        |

patch_format can include the following values:

  | Value | Patch Format             |
  | ----- | ------------------------ |
  | 0     | Brotli Shared Dictionary |

## Response

  | ID | Field Name             | Type                 |
  | -- | ---------------------- | -------------------- |
  | 0  | response_type          | UIntBase128          |
  | 1  | original_font_checksum | Uint64               |
  | 2  | patch_format           | UIntBase128          |
  | 3  | patch                  | ArrayOf<byte>        |
  | 4  | patched_checksum       | Uint64               |
  | 5  | codepoint_ordering     | CompressedList       |
  | 5  | ordering_checksum      | Uint64               |

response_type can be one of the following values:

  | Value | Response Type            |
  | ----- | ------------------------ |
  | 0     | PATCH                    |
  | 1     | REBASE                   |
  | 2     | REINDEX                  |
  

## CompressedList

  | ID | Field Name             | Type                 |
  | -- | ---------------------- | -------------------- |
  | 0  | value_deltas           | ArrayOf<IntBase128>  |
  | 1  | range_deltas           | ArrayOf<UIntBase128> |

## CompressedSet

  | ID | Field Name             | Type                 |
  | -- | ---------------------- | -------------------- |
  | 0  | sparse_bit_set         | ArrayOf<Uint8>       |
  | 1  | range_deltas           | ArrayOf<UIntBase128> |

# Request Behaviour

# Response Behaviour
