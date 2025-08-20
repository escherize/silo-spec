# Tortise File Format Specification v0.1

## 1. Purpose
A single UTF-8 plaintext file that represents a complete directory tree. Optimized for hand-writing and inspection.

## 2. Terminology
Key words **MUST**, **SHOULD**, **MAY** are per RFC 2119/8174.

## 3. Overview
A Tortise file is a sequence of file sections. Each section starts with a **file declaration** line, followed by the file's **content** lines. A fixed **delimiter** string identifies declaration lines. Blank lines outside sections are ignored.

## 4. Delimiter
- The delimiter **MUST** be a sequence of one or more ASCII punctuation characters (e.g., `>`, `===`, `***`, `->`, `#!`).
- The delimiter is **auto-detected** from the first non-blank line, which **MUST** be a valid file declaration.
- All declarations in the file **MUST** use the **exact same delimiter string**.
- Writers **MUST** choose a delimiter such that no content line in the file begins with `<DELIM><SP>`. If a collision would occur, choose a different delimiter.

## 5. File declaration
Syntax:
```
<DELIM><SP><PATH><EOL>
```
Rules:
- `<SP>` is one ASCII space (0x20).
- `<PATH>` is a POSIX-style relative path using `/`.
- Paths **MUST NOT** be absolute, contain `..`, or contain NUL.
- Duplicate `<PATH>`s **MUST** be rejected.

## 6. Content
- All lines after a declaration up to the next declaration belong to that file.
- Content is included **verbatim**.
- Each file's content **MUST** end with a newline. Readers **MUST** behave as if a trailing `\n` is present. Writers **SHOULD** include it.
- Readers **MUST** accept `\n` and `\r\n`; delivered content uses `\n`.

## 7. Collisions
- A line starting with `<DELIM><SP>` is always interpreted as a file declaration.
- Writers **MUST** choose a delimiter such that no content line begins with `<DELIM><SP>`.
- If a collision would occur, writers **MUST** choose a different delimiter.

## 8. Ignored lines
- Blank or whitespace-only lines **outside** any section are ignored.
- Blank lines **inside** a section are preserved.

## 9. Directories
- Directories are implicit. Implementations **MUST** create intermediate directories as needed for each `<PATH>`.

## 10. Media type and extension
- Suggested media type: `application/vnd.tortise.tree`
- Suggested extension: `.tortise`

## 11. Security considerations
- Reject absolute paths, `..`, and drive letters.
- Impose limits (file count, path length, file size).
- Write atomically with safe permissions.

## 12. Errors
Implementations **MUST** error on:
- Inconsistent delimiter string.
- Duplicate `<PATH>`.
- Disallowed `<PATH>` (absolute, `..`, empty, or `.`).

## 13. ABNF (informative)
ABNF cannot enforce "same delimiter string" globally; prose above is normative.
```abnf
tortise         = *blank-line *(section *blank-line)
section         = file-decl *content-line
file-decl       = delim SP path-chars EOL
delim           = 1*punct-char  ; some non-empty punct-char sequence
punct-char      = "!" / "#" / "$" / "%" / "&" / "'" / "(" / ")" / "*" / 
                  "+" / "," / "-" / "." / "/" / ":" / ";" / "<" / "=" / 
                  ">" / "?" / "@" / "[" / "\" / "]" / "^" / "_" / "`" / 
                  "{" / "|" / "}" / "~"
path-chars      = 1*path-char
path-char       = %x21-7E except %x20  ; visible ASCII except space
content-line    = *VCHAR EOL
blank-line      = *WSP EOL
SP              = %x20
EOL             = %x0A / %x0D.0A
VCHAR           = %x21-7E
WSP             = %x20 / %x09
```

## 14. Reference parsing algorithm (normative)
```
normalize EOL to "\n"; read UTF-8
skip leading blank/whitespace-only lines
read first non-blank line L
  parse DELIM as the longest prefix of punctuation chars such that
    L == DELIM + " " + PATH where PATH is non-empty
  require |DELIM| >= 1
current = new file(PATH)
for each subsequent line:
  if line starts with DELIM + " ":
     finalize current; start new file(PATH = text after space)
  else:
     append line to current file content verbatim
on close of a section, ensure content ends with "\n"
validate paths and duplicates
```

## 15. Writer guidelines
- Prefer `>` as the delimiter. If any content would start with `> `, switch to `===`, `***`, `->`, or another punctuation sequence.
- End the container file with `\n`.

## 16. Example
Using `===` to avoid a `> ` collision in content:
```
=== src/util.py
a = 1

=== hi.py
from src.util import a
print(a)
> this line starts with >

=== config/settings.json
{
  "debug": true
}
```

Yields three files:
- `src/util.py` with content `a = 1\n`
- `hi.py` with content:
  ```
  from src.util import a
  print(a)
  > this line starts with >
  ```
- `config/settings.json` with content `{\n  "debug": true\n}\n`

## 17. Conformance
A "Tortise reader" **MUST** implement Sections 4–12. A "Tortise writer" **MUST** implement Sections 4–8 and **SHOULD** follow Section 15.
