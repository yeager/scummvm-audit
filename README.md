# ScummVM Security & Quality Audit

> ⚠️ **Work in progress** — None of this has been tested. The patches were produced through static analysis and code review but have not been compiled or run against ScummVM. Use at your own risk.

Patches addressing security vulnerabilities, memory safety issues, and code quality improvements found in ScummVM.

## Findings Summary

| Category | Count | Severity |
|----------|-------|----------|
| Buffer overflow (strcpy/sprintf) | ~390 files | High |
| Format string (sprintf without bounds) | ~354 files | Medium-High |
| Unchecked input parsing (atoi/sscanf) | ~432 files | Medium |
| Missing null checks (delete/free) | ~1150 files | Medium |
| Integer overflow in allocation | Multiple | High |
| strncpy without null-termination | Multiple | Medium |

## Patches

### 001-twp-lip-sscanf-return-check.patch
**Problem:** `engines/twp/lip.cpp` calls `sscanf()` without checking the return value. Malformed .lip files with invalid data will cause `item.time` and `item.letter` to contain uninitialized values, leading to undefined behavior.

### 002-drascula-sscanf-buffer-overflow.patch
**Problem:** `engines/drascula/resource.cpp` uses `sscanf(buf, "%s", result)` which has no bounds checking — the `%s` specifier writes unlimited data into `result`, causing a classic buffer overflow.

### 003-glk-utils-atoi-validation.patch
**Problem:** `engines/glk/utils.cpp` uses `atoi()` which silently returns 0 on invalid input and has undefined behavior on overflow. Replace with `strtol()` with proper error checking.

### 004-icb-px-string-bounds-check.patch
**Problem:** `engines/icb/common/px_string.cpp` uses `strcpy` (via `Common::strcpy_s`) without validating source string length against destination buffer size. The `_s` variant helps but callers may still overflow if buffer sizes are wrong.

### 005-bagel-boflib-log-sprintf.patch
**Problem:** `engines/bagel/boflib/log.h` describes its function as "builds a string like sprintf()" — uses unbounded string formatting that can overflow fixed-size log buffers.

### 006-agi-lzw-integer-overflow.patch
**Problem:** `engines/agi/lzw.cpp` uses `malloc(count * sizeof(...))` pattern where `count * sizeof(...)` can overflow on 32-bit systems, leading to heap corruption via undersized allocation.

### 007-cge-talk-allocation-overflow.patch
**Problem:** `engines/cge/talk.cpp` has a similar integer overflow risk in allocation — multiplying user-controlled sizes without overflow checking.

## Roadmap

- [ ] More `sprintf` → `snprintf` conversions (~354 files identified)
- [ ] `strcpy` → `Common::strlcpy` hardening across engine parsers
- [ ] `atoi` → `strtol` with validation in remaining engines
- [ ] Null-check audit for `delete`/`free` paths (~1150 files)
- [ ] `strncpy` null-termination guarantees
- [ ] Fuzz testing harness for file format parsers (`.lip`, `.drascula`, `.lzw`)
- [ ] Upstream PR submission to ScummVM

## Methodology

- GitHub code search API for known unsafe patterns
- Static analysis of security-critical code paths
- Focus on engine code that parses external file formats (highest attack surface)

## Author

Daniel Nylander <daniel@danielnylander.se>

## License

Patches are submitted under GPL-3.0-or-later, matching ScummVM's license.
