# BitFields
Stores and retrieves user-defined bit-width values in standard Byte[] form


A lightweight C++ bit-packing utility for storing multiple user-defined integer fields in a compact, contiguous byte array.

`BitField` allows fields of different widths and signedness to share the same storage, minimizing memory usage while providing simple indexed access through `setValue()` and `getValue()`.

Designed for embedded systems and Arduino-based applications where memory efficiency and predictable byte storage are important.

## Features

- Variable-width fields from 1 to 16 bits (with the current `int` interface).
- Independently configurable signed and unsigned fields.
- Signed values use conventional two's-complement representation.
- Automatic bit packing across byte boundaries.
- Compact, contiguous byte-array storage.
- Simple indexed read/write interface.
- Storage can be loaded from or saved to EEPROM, flash, or other byte-addressable memory.

## Installation

Include `BitFields.h` in your project:

```cpp
#include "BitFields.h"
```

The header uses Arduino types and includes `TGTDB.h` for optional project debugging support.

`Config.h` is included automatically when available.

## Defining Fields

Field definitions are supplied as an integer array during construction.

- **Positive values:** Unsigned field widths.
- **Negative values:** Signed field widths, including one sign bit.

For signed fields, the absolute value specifies the total width, including the sign bit.

### Example

```cpp
const int Fields[] = {8, -16, 5, -11, 2};

BitField bf(Fields, 5);
```

This defines five fields:

| Index | Definition | Payload bits | Total bits | Type |
|---:|---:|---:|---:|---|
| 0 | 8 | 8 | 8 | Unsigned |
| 1 | -16 | 15 | 16 | Signed |
| 2 | 5 | 5 | 5 | Unsigned |
| 3 | -11 | 10 | 11 | Signed |
| 4 | 2 | 2 | 2 | Unsigned |
| | | | **42** | **Total** |

The resulting storage requirement is:

```cpp
bf.totalBytes();  // Returns 6
```

The final byte contains unused bits because storage is rounded up to the nearest whole byte.

## Reading and Writing Values

Two primary methods provide indexed access to the defined fields.

### setValue()

```cpp
void setValue(byte FieldIdx, int val);
```

Stores a value in the specified field.

```cpp
bf.setValue(0, 200);
bf.setValue(1, -1234);
bf.setValue(2, 17);
bf.setValue(3, -500);
bf.setValue(4, 3);
```

### getValue()

```cpp
int getValue(byte FieldIdx);
```

Retrieves the value stored in the specified field.

```cpp
int PumpSpeed = bf.getValue(0);
int Pressure  = bf.getValue(1);
int Mode      = bf.getValue(2);
int Offset    = bf.getValue(3);
int State     = bf.getValue(4);
```

Signed fields are automatically sign-extended when retrieved.

## Byte Storage

All packed data resides in the public byte array:

```cpp
byte buff[16];
```

The storage can be transferred to and from any byte-addressable memory.

### Loading Data

Example using EEPROM or another persistent memory interface:

```cpp
for (int i = 0; i < bf.totalBytes(); i++) {
    bf.buff[i] = PMEMREAD(StartAddr + i);
}
```

### Saving Data

```cpp
for (int i = 0; i < bf.totalBytes(); i++) {
    PMEMWRITE(StartAddr + i, bf.buff[i]);
}
```

This separation between field definitions and byte storage allows the same data layout to be used with different storage mechanisms.

## Bit Layout

Fields are packed consecutively without byte-alignment requirements.

- Fields may begin or end anywhere within a byte.
- Fields may span multiple bytes.
- Bits within each field are stored most-significant-bit first.
- The first field begins at bit 0 of `buff[0]`.

For example, three fields defined as `{3, 5, 4}` occupy 12 bits:

```text
buff[0]                         buff[1]
  Bit 7 6 5 4 3 2 1 0             Bit 7 6 5 4 3 2 1 0
       [ Field 1 ][ Field 2 ]       [ Field 3 ][ unused ]
```

The layout is deterministic as long as the same field definitions are used when writing and reading the data.

## API Reference

| Method / Member | Description |
|---|---|
| `BitField(defs, count)` | Constructs a bit-field layout |
| `setValue(idx, val)` | Stores a field value |
| `getValue(idx)` | Retrieves a field value |
| `totalBytes()` | Returns required storage bytes |
| `buff[]` | Public packed byte storage |

## Limitations

Current implementation limits:

- Maximum of 60 field definitions per instance.
- Maximum storage of 16 bytes (128 bits).
- `int` is used for value input and output; supported field widths should therefore not exceed the width of the platform's `int`.
- Field indices must reference an existing definition.
- Values must fit their assigned field width. Excess high bits are truncated.
- The caller is responsible for using matching field definitions when loading previously stored data.

**Note:** The constructor does not currently validate field widths or total storage against the 16-byte buffer. Keep the combined field width within 128 bits and use valid field definitions.

## Design Philosophy

`BitField` separates the definition of a data structure from its physical storage.

Rather than dedicating a byte or integer to every variable, developers define exactly how many bits each value requires. The resulting packed byte array can then be stored, transmitted, or reconstructed independently of the original C++ variables.

This makes `BitField` particularly useful for embedded applications, compact configuration records, communication protocols, and persistent device settings.

---
