# ABI Compatibility Matrix

## Struct Layout Rules

| Zig Type | C Equivalent | Fortran bind(C) | Size |
|----------|-------------|-----------------|------|
| extern struct | struct | type, bind(C) | Platform ABI |
| packed struct | __attribute__((packed)) | N/A | No padding |
| struct (default) | N/A | N/A | Zig-specific |

## Calling Convention

Fortran bind(C) uses the C calling convention:
- x86-64: first 6 integer args in RDI, RSI, RDX, RCX, R8, R9
- x86-64: first 8 float args in XMM0-XMM7
- Return: RAX (integer), XMM0 (float)
- Stack: 16-byte aligned

## Verification Pattern

```zig
comptime {
    // Ensure Zig struct matches Fortran type
    std.debug.assert(@sizeOf(FortranType) == expected_size);
    std.debug.assert(@offsetOf(FortranType, "field") == expected_offset);
}
```
