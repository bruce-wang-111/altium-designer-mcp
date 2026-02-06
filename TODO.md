# TODO: PcbLib Writer Bug Fixes

## Status: READY FOR TESTING

All format bugs identified by studying AltiumSharp (C#) and pyAltiumLib (Python) have been fixed.
Ready for manual testing with Altium Designer.

---

## Completed Tasks

### 1. Fix OLE Version

- [x] Changed to OLE v3 (512-byte sectors) in both PcbLib and SchLib

### 2. Fix `FileHeader` Stream

- [x] Changed from pipe-delimited key=value to binary version string
- [x] Format: `[string_len:4 LE][string_len:1]["PCB 6.0 Binary Library File"]`
- [x] Reader updated to handle both formats (backward compatible)

### 3. Fix `/Library/Data` Stream

- [x] Added `/Library` storage with Header and Data streams
- [x] Parameter block uses null-terminated encoding (`[block_len:4][params + \x00]`)
- [x] Reader parses `/Library/Data` for component ordering

### 4. Fix Per-Component Streams

- [x] `/{component}/Header` — exact primitive count (removed erroneous +1)
- [x] `/{component}/Parameters` — block-prefixed with null terminator
- [x] `/{component}/Data` — binary primitives
- [x] `/{component}/WideStrings` — block-prefixed, leading pipe, empty = `[2:4]["|" + \x00]`
- [x] `/{component}/UniqueIdPrimitiveInformation` — 0-based indexing, null-terminated records

### 5. Fix Model Data Stream

- [x] Records use leading `|` pipe character
- [x] Null terminator included in block length

### 6. Reader Backward Compatibility

- [x] `FileHeader` reader handles both binary and pipe-delimited formats
- [x] UniqueID reader auto-detects 0-based vs 1-based indexing
- [x] Null terminators stripped from UniqueID records before parsing

---

## Testing Required

- [ ] Create a new PcbLib with the MCP server
- [ ] Open in Altium Designer — verify it loads without errors
- [ ] Check footprint rendering in Altium
- [ ] Test round-trip: create in MCP, open in Altium, save, open in MCP

---

## Key Changes Summary

| Stream | Old (Broken) | New (Correct) |
|--------|--------------|---------------|
| OLE Version | V4 (4096-byte) | V3 (512-byte) |
| `/FileHeader` | Pipe-delimited key=value | Binary version string |
| `/Library/Data` params | No null terminator | `[block_len:4][params + \x00]` |
| `/{comp}/Header` | primitive_count + 1 | Exact primitive_count |
| `/{comp}/Parameters` | Raw text | Block-prefixed with null terminator |
| `/{comp}/WideStrings` (empty) | `[0:4]` | `[2:4]["\|" + \x00]` |
| UniqueID indexing | 1-based | 0-based |
| UniqueID records | No null terminator | Null-terminated, included in block length |
| Model Data records | No leading pipe | Leading `\|`, null in block length |

---

## Files Modified

- `src/altium/pcblib/mod.rs` — FileHeader, Library/Data, Parameters, reader fixes
- `src/altium/pcblib/writer.rs` — Header count, WideStrings, UniqueID, Model Data encoders
- `src/altium/pcblib/reader.rs` — UniqueID null stripping, 0/1-based auto-detection
- `src/altium/schlib/mod.rs` — OLE v3
- `docs/PCBLIB_FORMAT.md` — Updated format documentation
