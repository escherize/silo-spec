## v0.2 — 2025-08-20

### Changed
- **Delimiter scope:** From "ASCII punctuation" to **any non-whitespace Unicode sequence**; emoji permitted.
- **Detection rule:** Delimiter parsed as the bytes before the first ASCII space on the first non-blank line.
- **Matching semantics:** Delimiter equality is **byte-exact in UTF-8**.
- **ABNF:** Updated to describe Unicode delimiter in prose; ASCII-only production removed.
- **Reference parser:** Algorithm revised to reflect the new delimiter rules.

### Added
- **Example:** Uses `🐢` as a delimiter.

### Compatibility
- **Backward:** v0.1 files remain valid.
- **Forward risk:** v0.1 readers that assume ASCII-only delimiters will reject v0.2 files using Unicode delimiters.

### Guidance
- Writers must avoid content lines beginning with `<DELIM><SP>`.
- Prefer short delimiters; pick one based on collisions. See: https://github.com/escherize/tortise_go/blob/master/tortise_go.go#L159 for a reference implementation.
- End the container file with `\n`.
