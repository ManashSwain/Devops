# YAML Rules — Complete Reference

> **YAML** = *YAML Ain't Markup Language* (originally *Yet Another Markup Language*).
> It is a **human-readable data serialization format** used to describe configuration and structured data.
> Every rule in this document is demonstrated with a runnable example in [`examples.yml`](examples.yml).

YAML is the language of DevOps: Kubernetes manifests, Docker Compose files, GitHub Actions workflows, Ansible playbooks, Helm charts, GitLab CI pipelines, Prometheus rules, and cloud templates are all YAML.

---

## Table of Contents

**Fundamentals**
- [1. What YAML Is](#1-what-yaml-is)
- [2. File Basics](#2-file-basics)
- [3. The Golden Rules](#3-the-golden-rules)
- [4. Indentation](#4-indentation)
- [5. Comments](#5-comments)
- [6. Documents](#6-documents)

**Core Structures**
- [7. Mappings](#7-mappings)
- [8. Sequences](#8-sequences)
- [9. Nesting](#9-nesting)
- [10. Flow Style](#10-flow-style)

**Values**
- [11. Scalars](#11-scalars)
- [12. Quoting Rules](#12-quoting-rules)
- [13. Data Types](#13-data-types)
- [14. Block Scalars](#14-block-scalars)

**Advanced**
- [15. Anchors, Aliases and Merge Keys](#15-anchors-aliases-and-merge-keys)
- [16. Tags](#16-tags)
- [17. Directives](#17-directives)
- [18. Advanced Types and Complex Keys](#18-advanced-types-and-complex-keys)

**Context**
- [19. YAML 1.1 vs YAML 1.2](#19-yaml-11-vs-yaml-12)
- [20. YAML vs JSON](#20-yaml-vs-json)
- [21. Common Errors](#21-common-errors)
- [22. Best Practices](#22-best-practices)
- [23. Security](#23-security)
- [24. Tooling](#24-tooling)
- [25. Real-World Patterns](#25-real-world-patterns)
- [26. Cheat Sheet](#26-cheat-sheet)

---

# 1. What YAML Is

| Property | Description |
|---|---|
| **Type** | Data serialization format (not a programming language) |
| **Purpose** | Represent mappings, sequences, and scalar values |
| **Style** | Whitespace-sensitive, line-oriented, human-readable |
| **Extension** | `.yaml` (preferred by the spec) or `.yml` |
| **Encoding** | UTF-8 (UTF-16 / UTF-32 also allowed by the spec, UTF-8 is universal) |
| **Spec versions** | 1.0, 1.1, 1.2 (current: **1.2.2**) |

YAML is a **superset of JSON**: any valid JSON document is valid YAML (in YAML 1.2). YAML adds comments, anchors, multi-document streams, and a friendlier syntax.

```yaml
# This YAML ...
server:
  host: localhost
  ports: [80, 443]
```

```json
// ... is equivalent to this JSON
{ "server": { "host": "localhost", "ports": [80, 443] } }
```

---

# 2. File Basics

## Rules

1. A YAML file is a **stream of one or more documents**.
2. Encoding is **UTF-8** by default. A BOM may appear only at the start.
3. Line breaks may be `LF` (`\n`) or `CRLF` (`\r\n`); parsers normalize them.
4. **Spaces** are the only legal indentation character.
5. A file may start with `---` (document start marker) — optional but recommended.
6. Comments and blank lines are allowed anywhere outside scalars.

```yaml
# deployment.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
```

---

# 3. The Golden Rules

Memorize these ten; they prevent 95% of YAML errors.

| # | Rule | Wrong | Right |
|---|---|---|---|
| 1 | **Never use tabs for indentation** | `\tkey: value` | `  key: value` |
| 2 | **A colon must be followed by a space** for a mapping | `key:value` | `key: value` |
| 3 | **A dash must be followed by a space** for a sequence item | `-item` | `- item` |
| 4 | **Indent children deeper than their parent** | `a:\nb: 1` | `a:\n  b: 1` |
| 5 | **Be consistent within a block** (same number of spaces) | 2 then 4 spaces | 2 then 2 spaces |
| 6 | **Keys in a mapping must be unique** | `a: 1\na: 2` | one `a` |
| 7 | **Quote values that look like other types** | `version: 1.10` | `version: "1.10"` |
| 8 | **`#` starts a comment only after whitespace or at line start** | `a: b#c` is one string | `a: b # c` |
| 9 | **Case matters** — keys and boolean spellings are case-sensitive | `Name` ≠ `name` | pick one |
| 10 | **An alias must point to an anchor defined earlier in the same document** | `*x` before `&x` | anchor first |

---

# 4. Indentation

## Rules

- Indentation is made of **spaces only**. A tab used for indentation is a syntax error in the spec and in every mainstream parser.
- Indentation defines **structure**, not formatting. More indentation = deeper level.
- Siblings must be at the **same indentation level**.
- The width is your choice (2 spaces is the universal convention), but it must be **consistent within a block**.
- An item's content may start on the same line as `-` or on following lines, indented.
- YAML does **not** require a fixed width like Python — only *relative* indentation matters.

```yaml
# 2-space indentation — the standard
server:
  host: localhost
  ports:
    - 80
    - 443
```

```yaml
# Both of these are VALID for sequences under a key:
ports:
  - 80
  - 443
```

```yaml
ports:
- 80
- 443
```

The second style (dash at the parent's level) is legal and used by `kubectl` output, but **pick one style per file**.

## Nested sequence indentation

```yaml
matrix:
  - - 1
    - 2
  - - 3
    - 4
```

The first dash opens a sequence at column 2; the second dash opens a nested sequence without a space after the first dash except one.

## Block scalar indentation

Inside `|` and `>`, indentation is **content** and is determined by the first non-empty line (or by an explicit indentation indicator). See [§14](#14-block-scalars).

---

# 5. Comments

## Rules

1. A comment starts with `#`.
2. A `#` begins a comment **only at the start of a line or after whitespace**.
3. `#` inside a quoted scalar is always data.
4. Comments run to the **end of the line**; there are no block comments.
5. Comments are **discarded by the parser** — they are not part of the data.
6. A comment cannot appear inside a plain scalar if not preceded by whitespace.
7. To comment multiple lines, repeat `#` per line.

```yaml
# Full-line comment
key: value            # Trailing comment

url: http://x.com/#frag   # '#' is preceded by '/', so it is DATA
message: "a # b"          # quoted -> '#' is DATA
literal: a#b              # plain scalar -> 'a#b'

# Sequential lines are separate comments
# There is no /* block comment */ in YAML.
```

Common mistake:

```yaml
# Clear: separate the comment from the value with whitespace
value: hello # comment

# Valid but confusing: this is the single plain scalar "hello#world"
value: hello#world
```

---

# 6. Documents

## Rules

- `---` **starts** a document.
- `...` **ends** a document (rarely needed; useful to terminate a document whose trailing content could be ambiguous).
- A single file can hold **multiple documents** = a *stream*.
- The first `---` is optional; any later `---` is required.
- Anchors, aliases, and directives are scoped **per document**.
- A parser reads a stream document by document.
- `---` can be followed by content on the same line: `--- value`.
- `...` is optional at end of stream.

```yaml
---
kind: Deployment
metadata:
  name: web
...
---
kind: Service
metadata:
  name: web
...
```

In Kubernetes, multiple resources are commonly placed in one file separated by `---`.

## Document markers as separators

```yaml
first: document
---
second: document
---
- third
- document
```

> **Rule:** You cannot define an anchor in document 1 and use its alias in document 2.

---

# 7. Mappings

A **mapping** (dictionary, object, hash) is a set of `key: value` pairs.

## Rules

1. The separator is `: ` — a colon **followed by whitespace** (or end of line).
2. A key and value may be on the same line, or the value may be a nested block on following, deeper-indented lines.
3. Keys must be **unique** within a mapping. Duplicate keys are invalid per spec; parsers either error or silently keep the last one (behavior is implementation-defined — never rely on it).
4. Keys are **case-sensitive**.
5. Keys may be unquoted, quoted, or **complex** (using `?` / `:`).
6. Empty value means `null`: `key:` → `{key: null}`.
7. A mapping may be empty: `{}`.
8. Ordering is **not guaranteed by the spec**, although most implementations preserve insertion order.

```yaml
person:
  name: Alice
  age: 30
  active: true
```

## Colon spacing — the classic error

```yaml
# WRONG — no space after the colon means there is no key/value separator
name:Alice               # this is a single plain scalar, NOT a key/value pair

person: {name:Alice}     # flow mapping with the single key "name:Alice" and null value
```

```yaml
# RIGHT
name: Alice
person: {name: Alice}
```

## Keys that need quoting

```yaml
"key with spaces": value
"key:with:colon": value
"yes": quoted key        # otherwise YAML 1.1 may resolve the key as boolean true
"123": numeric-looking key
```

## Duplicate keys

```yaml
# INVALID per spec
server: a
server: b
```

`yamllint` flags this as `key-duplicates`; some parsers raise an error.

---

# 8. Sequences

A **sequence** (array, list) is an ordered collection.

## Rules

1. Each item starts with `- ` (dash + space).
2. Items may be scalars, mappings, or nested sequences.
3. Sibling items must be at the same indentation.
4. A sequence under a mapping key may be indented (standard) or at the same level as the key (legal).
5. An empty sequence is `[]`.
6. Flow style: `[a, b, c]`.
7. A sequence item may itself start a mapping on the same line.

```yaml
fruits:
  - apple
  - banana
  - cherry
```

```yaml
servers:
  - name: web-01
    ip: 10.0.0.11
  - name: web-02
    ip: 10.0.0.12
```

Each `-` introduces a mapping whose keys continue at the dash's content column.

## Nested sequences

```yaml
matrix:
  - - 1
    - 2
  - - 3
    - 4
```

## Mixed types

```yaml
mixed:
  - 1
  - true
  - "three"
  - null
  - {key: value}
  - [1, 2, 3]
```

## Empty and single-item

```yaml
empty: []
single:
  - only
```

> **Rule:** `-` alone (dash, then end of line) is a valid `null` item, but `-item` without a space is a plain string.

---

# 9. Nesting

Any node may contain any other node. Structure comes entirely from indentation (block style) or brackets (flow style).

```yaml
application:
  name: demo
  owner:
    team: platform
    contact:
      email: platform@example.com
      slack: "#platform"
  environments:
    - name: dev
      replicas: 1
      features:
        - logging
        - debug
    - name: prod
      replicas: 3
      features:
        - logging
```

## Rules

- A child block must be **more indented** than its parent key.
- A value on the same line as the key cannot also have a nested block on following lines.
- You cannot mix a scalar value and nested children for the same key:

```yaml
# INVALID — "value" then a nested block
key: value
  child: x
```

```yaml
# VALID
key:
  child: x
```

---

# 10. Flow Style

Flow style uses JSON-like delimiters. It is compact and valid anywhere block style is.

## Rules

- Sequence: `[ item, item ]`
- Mapping: `{ key: value, key: value }`
- Elements separated by `,`.
- `key: value` still requires the space after the colon.
- Flow collections may span multiple lines; indentation inside is cosmetic.
- Flow style may be nested arbitrarily.
- Trailing commas are **not allowed** (unlike JSON5/JavaScript).
- In flow context, plain scalars cannot contain `,`, `[`, `]`, `{`, `}` without quoting.

```yaml
flow:
  list: [apple, banana, cherry]
  map: {name: Alice, age: 30, active: true}
  nested: {users: [{name: a}, {name: b}], count: 2}
  multiline: {
    name: Alice,
    age: 30,
    tags: [admin, dev]
  }
  empty_map: {}
  empty_list: []
```

> **Style rule:** use block style for configuration humans edit; flow style for short, simple values (e.g. `ports: [80, 443]`).

---

# 11. Scalars

A **scalar** is a single value: string, number, boolean, null, timestamp, binary.

## Plain scalars (unquoted)

A plain scalar may contain most characters, but has restrictions.

### Rules — a plain scalar **cannot**:

1. Start with an **indicator character**:
   `- ? : , [ ] { } # & * ! | > ' " % @ \``
   (with exceptions: `-`, `?`, `:` are allowed if *not* followed by whitespace).
2. Contain `: ` (colon + space) or ` #` (space + hash).
3. Contain `[ ] { } ,` in **flow context**.
4. Be empty (an empty value is `null`, not `""`).
5. Span multiple lines in block context, except by **folding** with indentation (still a single logical scalar).

```yaml
# VALID plain scalars
name: Alice
path: /usr/local/bin
url: http://example.com/path?q=1
version: 1.2.3
dash: hello-world
question: what?
colon_in_url: http://x.com:8080
```

```yaml
# INVALID or surprising plain scalars
# starts with indicator:
# @value
# `value
# value: a: b
# value: a # b        -> b becomes a comment
# value: [a, b]       -> flow sequence
```

## Multi-line plain scalars

Plain scalars can continue on following lines that are indented; line breaks fold to spaces.

```yaml
description: This is a long
  sentence that continues
  on other lines.
# value: "This is a long sentence that continues on other lines."
```

---

# 12. Quoting Rules

Quoting gives you **exact control** over the value and its type.

## When you MUST quote

| Situation | Example | Reason |
|---|---|---|
| Value would parse as another type | `"123"`, `"true"`, `"null"`, `"1.0"` | Keep it a string |
| Starts with an indicator | `"- item"`, `"@name"`, `"*alias"` | Otherwise parsed as syntax |
| Contains `: ` or ` #` | `"key: value"`, `"a # b"` | Otherwise split/comment |
| Leading or trailing whitespace | `" padded "` | Otherwise trimmed |
| Empty string | `""` | `key:` means null |
| YAML 1.1 booleans you mean literally | `"yes"`, `"no"`, `"on"`, `"off"`, `"NO"` | Otherwise becomes boolean |
| Numeric-looking versions / IDs | `"1.10"`, `"007"`, `"2024-01-01"` | Avoid semantic changes |
| Contains newlines you want escaped | `"line1\nline2"` | Control characters |
| Keys with special syntax | `"key:with:colon"` | Otherwise parsed as nested mapping |

## Single-quoted strings

- Everything is **literal**, except `''` which represents one `'`.
- No escape sequences are processed.

```yaml
text: 'It''s YAML'          # It's YAML
path: 'C:\Users\devops'     # backslashes preserved literally
```

## Double-quoted strings

- Process **escape sequences**.
- Support Unicode escapes.

| Escape | Meaning | Escape | Meaning |
|---|---|---|---|
| `\0` | NUL | `\n` | newline |
| `\t` | tab | `\r` | carriage return |
| `\"` | double quote | `\\` | backslash |
| `\b` | backspace | `\f` | form feed |
| `\v` | vertical tab | `\e` | escape |
| `\/` | slash | `\N` | next line |
| `\_` | non-breaking space | `\L` | line separator |
| `\P` | paragraph separator | `\xNN` | 8-bit char |
| `\uNNNN` | 16-bit Unicode | `\UNNNNNNNN` | 32-bit Unicode |

```yaml
escaped: "Line 1\nLine 2\tTabbed"
unicode: "caf\u00E9"                 # café
windows_path: "C:\\Users\\devops"
quote: "She said \"hello\""
```

## Quoting keys

Keys follow the same rules as values.

```yaml
"yes": 1          # literal string key
"key: with colon": 2
"": empty_key     # the empty string is a legal key
```

## Quotes are stripped

```yaml
a: "hello"
b: 'hello'
c: hello
# All three produce the string "hello".
```

---

# 13. Data Types

YAML resolves unquoted scalars to types using a **schema**. The default is the **Core Schema** (YAML 1.2), which recognizes **null, boolean, integer, float, string**. Tags such as `!!binary` and `!!timestamp` add more.

## 13.1 Null

| Form | Meaning |
|---|---|
| `~` | null |
| `null`, `Null`, `NULL` | null |
| *(empty value)* | null, e.g. `key:` |

```yaml
nothing1: ~
nothing2: null
nothing3:
empty_string: ""      # NOT null
```

## 13.2 Booleans

**YAML 1.2 Core Schema** (only these six spellings):

```yaml
a: true
b: True
c: TRUE
d: false
e: False
f: FALSE
```

**YAML 1.1** additionally treats these as booleans:

```yaml
# YAML 1.1 ONLY — strings in YAML 1.2
y: yes
n: no
on_value: on
off_value: off
y_short: y
n_short: n
```

> ⚠️ **Norway problem:** in YAML 1.1, the country code `NO` parses as `false`.
> Always quote `"NO"`, `"yes"`, `"on"`, etc. when you mean a string.

## 13.3 Integers

| Form | Example | Notes |
|---|---|---|
| Decimal | `42`, `-17`, `+5` | |
| Zero | `0` | |
| Hexadecimal | `0xFF`, `0x1a` | 1.1 & 1.2 |
| Octal | `0o755` | YAML **1.2** |
| Legacy octal | `0755` | YAML **1.1** only → `493` |
| Binary | `0b1010` | YAML **1.1**; not in 1.2 core schema |
| Underscores (not 1.2 core) | `1_000_000` | Many parsers accept it |

```yaml
decimal: 42
negative: -17
hex: 0xFF
octal: 0o755
binary: 0b1010
```

## 13.4 Floats

| Form | Example |
|---|---|
| Decimal point | `3.14`, `-0.5` |
| Exponent | `1.2e3`, `-4E-5` |
| Infinity | `.inf`, `.Inf`, `.INF`, `-.inf` |
| Not a number | `.nan`, `.NaN`, `.NAN` |

```yaml
pi: 3.14159
big: 1.2e+10
negative: -0.5
infinity: .inf
not_a_number: .nan
```

> A number without a decimal point is an **integer**. `1.0` is a float, `1` is an integer.
> Quote version strings: `version: "1.10"` — otherwise `1.10` becomes the float `1.1` and trailing zeros vanish in some serializers.

## 13.5 Strings

Anything quoted is a string. Unquoted text that matches no other type is a string.

```yaml
plain: hello
single: 'hello'
double: "hello"
number_like: "42"
bool_like: "true"
date_like: "2024-01-01"
```

## 13.6 Timestamps

The `!!timestamp` type (YAML 1.1 type repository) recognizes ISO-8601-like values:

```yaml
date: 2024-01-15
datetime: 2024-01-15T10:30:00Z
datetime_offset: 2024-01-15T10:30:00+05:30
```

> ⚠️ YAML **1.2 Core Schema does not resolve timestamps automatically** — they stay strings unless tagged `!!timestamp`. But many libraries (PyYAML, Ruby Psych, etc.) still resolve them to date objects. If you need a guaranteed string, quote it.

## 13.7 Binary

Base64 content tagged `!!binary`:

```yaml
certificate: !!binary |
  TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQkFRQXd
```

## 13.8 Type summary table

| Type | Tag | Example |
|---|---|---|
| Null | `!!null` | `~` |
| Boolean | `!!bool` | `true` |
| Integer | `!!int` | `42` |
| Float | `!!float` | `3.14` |
| String | `!!str` | `"hi"` |
| Mapping | `!!map` | `{a: 1}` |
| Sequence | `!!seq` | `[1, 2]` |
| Binary | `!!binary` | base64 |
| Timestamp | `!!timestamp` | `2024-01-15` |

---

# 14. Block Scalars

Block scalars are for **multi-line strings**. Two indicators:

| Indicator | Name | Behavior |
|---|---|---|
| `\|` | **Literal** | Keeps line breaks exactly |
| `>` | **Folded** | Folds single line breaks into spaces |

## 14.1 Literal `|`

```yaml
script: |
  #!/bin/bash
  echo "hello"
  exit 0
```

The value preserves newlines and trailing newline.

## 14.2 Folded `>`

```yaml
description: >
  This is a long sentence
  that becomes one line.
```

Value: `"This is a long sentence that becomes one line.\n"`

### Folding rules

| Input pattern | Result |
|---|---|
| Single line break between lines | replaced with a space |
| Two consecutive line breaks (blank line) | one newline |
| A **more-indented** line | kept literally, newlines preserved around it |
| Leading/trailing blank lines | subject to chomping |

## 14.3 Chomping indicators

Placed **after** the indicator:

| Form | Name | Trailing newline |
|---|---|---|
| `\|` or `>` | **Clip** (default) | exactly one newline kept |
| `\|-` or `>-` | **Strip** | no trailing newline |
| `\|+` or `>+` | **Keep** | all trailing newlines kept |

```yaml
clip: |
  line
  # -> "line\n"

strip: |-
  line
  # -> "line"

keep: |+
  line


  # -> "line\n\n\n" (all trailing blank lines kept)
```

## 14.4 Indentation indicator

An explicit digit declares the content indentation **relative to the parent node**:

```yaml
code: |2
    indented two extra spaces
```

The `2` means content starts 2 spaces beyond the key's indentation; the remaining spaces are content.

> **Rule:** the indentation indicator is only needed when the first line starts with extra spaces (otherwise indentation is auto-detected from the first non-empty line). Auto-detection can misread code blocks, so `|2` / `|4` is common for scripts.

## 14.5 Rules

1. The indicator (`|` / `>`) must be on the same line as the mapping key (or the sequence dash).
2. Content must be indented **more** than the parent.
3. Content indentation is set by the **first non-empty line** unless an explicit indicator is given.
4. Blank lines are always preserved as newlines (subject to chomping).
5. A line that is **less indented** ends the block.
6. Tabs inside content are allowed (only *indentation* forbids tabs).
7. You cannot use block scalars in flow style.

---

# 15. Anchors, Aliases and Merge Keys

Anchors and aliases are YAML's **reuse mechanism** — define once, reference many times.

## Rules

1. `&name` **defines** an anchor on a node.
2. `*name` **references** (aliases) the anchored node.
3. The anchor must be defined **before** the alias, in the **same document**.
4. An alias refers to the **same node/value**, not a deep copy (implementations may share the object).
5. Anchor names may contain letters, digits, `-`, `_`, and most non-space characters; they cannot contain `[ ] { } , & *` and whitespace.
6. You cannot change an anchored node through the alias in most parsers (immutable sharing).
7. Anchors can be placed on scalars, sequences, and mappings.
8. An anchor definition itself does not add data — the node still appears at that position.

```yaml
defaults: &defaults
  adapter: postgres
  host: localhost
  port: 5432

development:
  <<: *defaults
  database: dev_db

production:
  <<: *defaults
  host: prod-db.internal
  database: prod_db
```

## Scalar and sequence anchors

```yaml
timeout: &timeout 30
service_a:
  timeout: *timeout
service_b:
  timeout: *timeout

common_ports: &ports [80, 443]
web:
  ports: *ports
```

## Merge keys `<<`

> **Note:** The merge key is officially a YAML **1.1** feature, *not* part of the YAML 1.2 Core Schema. It is nonetheless supported by PyYAML, Ruby Psych, Docker Compose, Ansible, and most DevOps tools. If portability matters, prefer anchors without merge or confirm support.

### Merge rules

1. `<<: *anchor` merges a **mapping** into the current mapping.
2. `<<: [*a, *b]` merges several mappings **in order**: **earlier entries win** over later ones.
3. **Explicit keys always override merged keys**, regardless of order.
4. The anchor must resolve to a mapping.
5. You may use merge multiple times in one mapping (later merges override earlier ones).
6. The merged keys become normal keys in the result.

```yaml
base: &base
  a: 1
  b: 2

override: &override
  b: 20
  c: 30

combined:
  <<: [*base, *override]   # a=1 (base), b=2 (base wins), c=30
  d: 40
```

## Aliases in sequences

```yaml
users:
  - &alice
    name: Alice
    role: admin
  - name: Bob
    role: dev
  - *alice            # exact same mapping again
```

> **Warning:** aliases can create very large structures when reused in many places (an "expansion bomb"). Modern parsers cap alias expansion.

---

# 16. Tags

A **tag** explicitly declares a node's type or attaches application semantics.

## 16.1 Standard (Core) tags

| Tag | Meaning | Example |
|---|---|---|
| `!!str` | string | `!!str 123` → `"123"` |
| `!!int` | integer | `!!int "42"` → `42` |
| `!!float` | float | `!!float "3.14"` |
| `!!bool` | boolean | `!!bool "true"` |
| `!!null` | null | `!!null ""` |
| `!!map` | mapping | `!!map {a: 1}` |
| `!!seq` | sequence | `!!seq [1, 2]` |
| `!!binary` | base64 binary | `!!binary \| ...` |
| `!!timestamp` | date/time | `!!timestamp 2024-01-15` |

```yaml
explicit_string: !!str 123        # "123" (string, not int)
explicit_int: !!int "42"          # 42
explicit_float: !!float "3.14"    # 3.14
explicit_bool: !!bool "true"      # true
explicit_null: !!null ""          # null
```

## 16.2 Verbatim tags

```yaml
value: !<tag:yaml.org,2002:str> 123
```

The `!!` prefix is shorthand for `!<tag:yaml.org,2002:...>`.

## 16.3 Custom / application tags

```yaml
config: !myapp/config
  retries: 3
  timeout: 10
```

- `!foo` declares a **local tag**.
- Custom tags require application support; generic parsers may reject or ignore them.
- In Python, `yaml.safe_load` raises on unknown tags unless you register constructors.
- In Kubernetes/Compose, custom tags generally are not allowed.

## 16.4 The non-specific tag `!`

`! value` is the **non-specific tag**. The YAML spec resolves it to `!!str`, so `! 123` is the string `"123"` — but parser support varies (older Ruby Psych resolves it as an integer), so prefer `!!str` when correctness matters.

```yaml
value: ! 123      # "123" per spec; !!str is the portable spelling
```

## Rules

1. A tag is written **before** the node: `!!int 42`.
2. Tags apply to the **whole node** (map/seq/scalar).
3. Anchors and tags can combine: `&anchor !!str value` (anchor usually first).
4. Unknown tags are implementation-defined: error, ignore, or custom-construct.
5. Tags cannot be used to bypass indentation rules.

---

# 17. Directives

Directives appear at the **very top of a document**, before `---`, and apply to that document.

## 17.1 `%YAML` — version directive

```yaml
%YAML 1.2
---
key: value
```

Rules:
- Must be followed by `---`.
- At most **one** `%YAML` per document.
- Parsers may reject unsupported versions.
- Exact string `1.2` or `1.1`.

## 17.2 `%TAG` — tag shorthand directive

```yaml
%TAG !e! tag:example.com,2000:app/
---
!e!config
  retries: 3
```

Rules:
- Syntax: `%TAG <handle> <prefix>`.
- Handle starts with `!` and ends with `!` (e.g. `!e!`); `!` and `!!` are reserved.
- Lets you write `!e!config` instead of `!<tag:example.com,2000:app/config>`.
- Applies only to the following document.
- Multiple `%TAG` directives allowed.
- Directives are separated from content by `---`.

> **Caution:** Directives are the least portable part of YAML. Most DevOps tools (Kubernetes, Compose) do not accept custom directives/tags. Use them only when the consuming application supports them.

---

# 18. Advanced Types and Complex Keys

## 18.1 Explicit keys (`?` and `:`)

For keys that are too long or are collections, use explicit key syntax.

```yaml
? [host, port]
: "a composite key"
```

Equivalent flow form:

```yaml
{? [host, port]: "a composite key"}
```

Another example:

```yaml
? |
  multi-line
  key
: value
```

Rules:
- `?` introduces a key at a deeper indentation; `:` introduces its value.
- Keys may be mappings/sequences, but the **target language must support unhashable keys** — Python's `dict` does not, so this is parsed then fails on construction.
- Explicit keys are rarely used in DevOps configuration.

## 18.2 Sets (`!!set`)

A set is a mapping whose values are null.

```yaml
environments: !!set
  ? development
  ? staging
  ? production
```

## 18.3 Ordered maps (`!!omap`)

A sequence of single-pair mappings that preserves order.

```yaml
steps: !!omap
  - build: "compile the app"
  - test: "run the tests"
  - deploy: "ship it"
```

## 18.4 Pairs (`!!pairs`)

A sequence of single-pair mappings where duplicate keys are allowed.

```yaml
headers: !!pairs
  - Accept: text/html
  - Accept: application/json
```

## 18.5 Binary (`!!binary`)

```yaml
icon: !!binary |
  iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk
```

## 18.6 Empty nodes

```yaml
empty_map: {}
empty_seq: []
empty_string: ""
empty_value:          # null
```

> **Rule:** `!!set`, `!!omap`, `!!pairs` are YAML **1.1** types. YAML 1.2 does not define them in the core schema; support varies by parser. Avoid them in portable configuration.

---

# 19. YAML 1.1 vs YAML 1.2

This matters constantly in DevOps because tools embed different parsers.

| Feature | YAML 1.1 | YAML 1.2 (Core) |
|---|---|---|
| `yes` / `no` / `on` / `off` as booleans | ✅ yes | ❌ strings |
| `y` / `n` as booleans | ✅ | ❌ |
| Octal `0777` | ✅ | ❌ (must be `0o777`) |
| Octal `0o777` | ❌ | ✅ |
| Binary `0b1010` | ✅ | ❌ in core schema |
| Sexagesimal `1:30` = 90 | ✅ | ❌ |
| Merge key `<<` | ✅ type | ❌ core (widely supported anyway) |
| `!!timestamp` auto-resolution | ✅ | ❌ core (library-dependent) |
| Superset of JSON | ⚠️ mostly | ✅ fully |
| Tab indentation | ❌ | ❌ |

## The Norway problem

```yaml
country: NO            # YAML 1.1 -> false (boolean!)
country_fixed: "NO"    # always the string "NO"
```

## Version-pinning

- GitHub Actions uses a YAML 1.2-style parser, so `on:` is a string key.
- Docker Compose and Ansible are based on PyYAML (YAML 1.1), so `on`, `yes`, `no` may become booleans.
- Kubernetes uses `sigs.k8s.io/yaml`, which converts YAML → JSON with YAML 1.1-ish rules; quote ambiguous values.

```yaml
# Safe in every parser
enabled: "yes"      # if you truly need the string "yes"
enabled_bool: true  # if you need a boolean
```

---

# 20. YAML vs JSON

| Aspect | YAML | JSON |
|---|---|---|
| Readability | High | Medium |
| Comments | ✅ | ❌ |
| Anchors/aliases | ✅ | ❌ |
| Multi-document | ✅ | ❌ |
| Strictness | Forgiving, type ambiguity | Strict, explicit |
| Whitespace | Significant | Insignificant |
| Trailing commas | ❌ | ❌ |
| Quoted keys | Optional | Required |
| Data model | Superset (1.2) | Subset |
| Best for | Human-edited config | Machine interchange |

Any JSON document is valid YAML 1.2. The reverse is not true: YAML's anchors, comments, and unquoted keys have no JSON equivalent.

```yaml
# Valid JSON is also valid YAML
{"name": "Alice", "age": 30, "tags": ["admin"]}
```

---

# 21. Common Errors

| # | Error | Bad | Good | Why |
|---|---|---|---|---|
| 1 | Tab indentation | `→key: value` | `  key: value` | Tabs are illegal for indentation |
| 2 | Missing space after colon | `key:value` | `key: value` | Becomes one plain scalar |
| 3 | Missing space after dash | `-item` | `- item` | Becomes a plain string |
| 4 | Inconsistent indentation | 2 spaces then 4 | 2 then 2 | Breaks nesting |
| 5 | Duplicate keys | two `name:` | one key | Invalid / last-wins |
| 6 | Unquoted `yes`/`no` | `country: NO` | `country: "NO"` | Norway problem |
| 7 | Unquoted version | `version: 1.10` | `version: "1.10"` | Float → `1.1` |
| 8 | `#` used in a value | `color: #fff` | `color: "#fff"` | Starts a comment |
| 9 | Colon in unquoted string | `cmd: echo a: b` | `cmd: "echo a: b"` | Splits into mapping |
| 10 | Alias before anchor | `*x` first | `&x` first | Alias must resolve |
| 11 | Alias across documents | anchor in doc 1 | same doc | Anchors are per-document |
| 12 | Block scalar at wrong indent | content at key level | content deeper | Ends the block |
| 13 | Trailing comma in flow | `[a, b,]` | `[a, b]` | Not allowed |
| 14 | Unquoted `{`/`[` in text | `desc: {urgent}` | `desc: "{urgent}"` | Parsed as flow mapping |
| 15 | Empty string written as nothing | `name:` | `name: ""` | `name:` is null |
| 16 | Wrong file extension | `.yamll` | `.yaml` / `.yml` | Tools ignore it |
| 17 | Tabs in block scalar content | allowed! | — | Tabs are legal in *content* |
| 18 | Assuming map order | relying on order | use a list | Order is not guaranteed by spec |

## Example: the two most common mistakes

```yaml
# WRONG
apiVersion: v1
kind: Pod
metadata:
    name: web
  labels:        # indentation breaks here
    app: web
```

```yaml
# RIGHT
apiVersion: v1
kind: Pod
metadata:
  name: web
  labels:
    app: web
```

---

# 22. Best Practices

## Formatting

1. Use **2-space indentation**, never tabs.
2. Keep lines under ~120 characters (yamllint `line-length`).
3. Start Kubernetes/multi-resource files with `---`.
4. Use one style per file: block style for structure, flow style only for short lists/maps.
5. Indent sequence items under their key (`  - item`, not pulling dashes to the key column) unless matching `kubectl` output.
6. Add a space before inline comments; align comments where it improves readability.
7. End the file with a single newline.

## Correctness

8. **Quote ambiguous scalars**: versions, ports-as-strings, IDs, `yes/no/on/off`, leading zeros, dates.
9. Avoid duplicate keys; enable `key-duplicates` linting.
10. Prefer explicit booleans `true`/`false` over `yes`/`no`.
11. Use `"..."` for values containing `: `, ` #`, or starting with an indicator.
12. Don't rely on mapping order; use a sequence when order matters.
13. Avoid sexagesimal-looking values (`1:30`) and legacy octal (`0755`).
14. Validate with the target tool, not just a generic parser (`kubectl apply --dry-run=client -f`, `docker compose config`, `ansible-lint`).

## Reuse

15. Use anchors/aliases for repeated blocks; keep runbooks aware that merge keys are 1.1.
16. Prefer one document per file for Kubernetes resources unless they are tightly coupled (Deployment + Service).
17. Keep YAML declarative; avoid embedding complex logic.

## Naming

18. Use `kebab-case` or `snake_case` consistently for keys (match the consuming tool's schema).
19. Use `.yaml` for new files; keep `.yml` only where a tool requires it.
20. Name files by resource: `deployment.yaml`, `values.yaml`, `docker-compose.yml`.

---

# 23. Security

YAML is data, but careless parsing can execute code or exhaust memory.

## 23.1 Arbitrary object construction

Some libraries historically allowed tags to instantiate arbitrary language objects.

```python
# DANGEROUS — Python (PyYAML < 5.1 default, or explicit UnsafeLoader)
yaml.load(untrusted_data)          # can execute code via python/object tags

# SAFE
yaml.safe_load(untrusted_data)
```

Rules:
- **Never** parse untrusted YAML with an unsafe loader.
- Allowlist tags if custom types are required.
- In Ruby, prefer `YAML.safe_load` with permitted classes.
- In JavaScript, use schema-safe parsers; avoid constructing functions from tags.
- Treat YAML from pull requests, user uploads, or third-party repos as untrusted.

## 23.2 Alias expansion / "billion laughs"

Recursive aliases can expand exponentially.

```yaml
a: &a ["x", "x", "x", "x", "x", "x", "x", "x", "x"]
b: &b [*a, *a, *a, *a, *a, *a, *a, *a, *a]
c: &c [*b, *b, *b, *b, *b, *b, *b, *b, *b]
# ... continues -> enormous memory use
```

Rules:
- Limit alias expansion (`max_aliases` in parsers that support it).
- Reject YAML with excessive anchors/aliases from untrusted sources.
- Enforce input size limits in APIs that accept YAML.

## 23.3 Secrets in YAML

- Do not commit plaintext secrets in Kubernetes manifests, Compose files, or workflows.
- Quote/escape is **not** encryption.
- Use Secret objects, external secret managers, CI secret stores, or SOPS/Sealed Secrets.
- Beware of `echo`-ing secrets in YAML-driven CI logs.

## 23.4 Denial of service and portability

- Deeply nested structures can hit parser depth limits.
- Tags like `!!binary` can smuggle large payloads.
- Different parsers resolve types differently (see §19) — validate with the actual consumer.

---

# 24. Tooling

| Tool | Purpose | Example |
|---|---|---|
| **yamllint** | Style and correctness linting | `yamllint -d relaxed file.yaml` |
| **yq** | Query/edit YAML like jq | `yq '.spec.replicas' deploy.yaml` |
| **kubectl** | Validate Kubernetes YAML | `kubectl apply --dry-run=client -f file.yaml` |
| **kubeconform** | Kubernetes schema validation | `kubeconform -strict deploy.yaml` |
| **docker compose config** | Validate & render Compose | `docker compose config` |
| **ansible-lint** | Ansible playbook linting | `ansible-lint site.yml` |
| **actionlint** | GitHub Actions linting | `actionlint .github/workflows/*.yml` |
| **prettier** | Format YAML | `prettier --write file.yaml` |
| **Python** | Parse safely | `yaml.safe_load(open("f.yaml"))` |
| **Ruby** | Parse | `YAML.safe_load(File.read("f.yaml"), permitted_classes: [Date])` |
| **VS Code YAML extension** | Schema-aware editing | Red Hat YAML extension |

## Minimal yamllint config

```yaml
# .yamllint
extends: default
rules:
  line-length:
    max: 120
  indentation:
    spaces: 2
    indent-sequences: true
  truthy:
    allowed-values: ["true", "false"]
  comments:
    min-spaces-from-content: 1
```

## Pre-commit hook

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/adrienverge/yamllint
    rev: v1.35.1
    hooks:
      - id: yamllint
```

---

# 25. Real-World Patterns

## 25.1 Kubernetes manifest

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
          env:
            - name: LOG_LEVEL
              value: "info"
          resources:
            limits:
              cpu: "500m"
              memory: "256Mi"
```

Key YAML lessons here:
- `apiVersion`, `kind`, `metadata`, `spec` are **fixed schema keys**.
- Everything under `metadata` and `spec` is a mapping; containers is a **sequence of mappings**.
- Resource values like `"500m"` and `"256Mi"` are **strings** — quote them.
- Multiple resources separated by `---`.

## 25.2 Docker Compose

```yaml
services:
  web:
    image: nginx:1.27
    ports:
      - "8080:80"
    environment:
      - APP_ENV=production
    depends_on:
      - db
    volumes:
      - ./html:/usr/share/nginx/html:ro

  db:
    image: postgres:16
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: change_me

volumes:
  db-data:
```

Lessons:
- Port mappings are quoted strings (`"8080:80"`) — a bare `8080:80` is a **sexagesimal number** in YAML 1.1!
- Environment variables can be a list or a mapping.
- Top-level `volumes:` with no value defines a named volume as `null`.

## 25.3 GitHub Actions

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: npm
      - run: npm ci
      - run: npm test
```

Lessons:
- `on:` is a **key**, not a boolean — GitHub's parser treats it as a string (YAML 1.2). Some generic YAML 1.1 parsers would read `on` as `true`.
- Pin action versions with `@v4`.
- `node-version: "20"` is quoted because `20` would be an integer.

## 25.4 Ansible playbook

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true

  vars:
    http_port: 80
    app_user: deploy

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Deploy configuration
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        mode: "0644"
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

Lessons:
- The top-level document is a **sequence** of plays.
- Jinja2 templates use `{{ }}`; if a value **starts** with `{`, it is parsed as a flow mapping — quote the whole value.
- `mode: "0644"` must be quoted, otherwise YAML 1.1 may treat it as an octal integer.

## 25.5 Helm `values.yaml`

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "1.27"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  limits:
    cpu: "500m"
    memory: "256Mi"
  requests:
    cpu: "100m"
    memory: "128Mi"

nodeSelector: {}
tolerations: []
affinity: {}
```

Lessons:
- Values templates are pure YAML; Helm adds Go templating around it.
- Important: values starting with `{{` need quoting in *some* contexts — Helm processes templates before YAML parsing in files, but in `--set` and some fields quoting matters.

## 25.6 Multi-document file

```yaml
---
kind: Namespace
apiVersion: v1
metadata:
  name: demo
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: app-config
  namespace: demo
data:
  LOG_LEVEL: info
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: app
  namespace: demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
        - name: app
          image: nginx:1.27
```

---

# 26. Cheat Sheet

## Syntax

```yaml
key: value                 # mapping
- item                     # sequence item
key:                       # nested block
  child: value
key: [a, b]                # flow sequence
key: {a: 1, b: 2}          # flow mapping
key: "text"                # double-quoted string
key: 'text'                # single-quoted string
key: |                     # literal block
key: >                     # folded block
key: |-                    # literal, strip trailing newline
key: >+                    # folded, keep trailing newlines
&anchor value              # define anchor
*anchor                    # alias
<<: *anchor                # merge mapping
!!str 123                  # explicit tag
? complex                  # explicit key
: value
---                        # document start
...                        # document end
# comment                  # comment
```

## Type resolution (YAML 1.2 Core)

| Looks like | Becomes |
|---|---|
| `null`, `~`, empty | null |
| `true` / `false` (any case) | boolean |
| `42`, `-7`, `0xFF`, `0o755` | integer |
| `3.14`, `1e3`, `.inf`, `.nan` | float |
| anything else | string |

## Must-quote checklist

- `yes` / `no` / `on` / `off` / `y` / `n` when literal
- `NO` (Norway)
- Versions: `"1.0"`, `"1.10"`
- Leading zeros: `"007"`
- Ports / sexagesimal: `"8080:80"`
- Values with `: ` or ` #`
- Values starting with `- ? : , [ ] { } # & * ! | > ' " % @ \``
- Empty string `""`
- Kubernetes quantities: `"500m"`, `"256Mi"`
- File modes: `"0644"`
- Dates you want as strings: `"2024-01-15"`

## The five-second validator

```bash
python3 -c "import yaml,sys; yaml.safe_load(open(sys.argv[1]))" file.yaml
# or
ruby -ryaml -e 'YAML.safe_load(File.read(ARGV[0]), permitted_classes: [Date, Time])' file.yaml
# or
yamllint file.yaml
```

---

# 27. Summary

| Layer | You must remember |
|---|---|
| **Declarative YAML** | Its name says "Ain't Markup Language": YAML describes data, not presentation |
| **Core** | Indentation with spaces; `key: value`; `- item`; nesting by indentation; unique keys |
| **Scalars** | Plain vs quoted; quote anything ambiguous; three string styles; block scalars for multi-line |
| **Types** | null / bool / int / float / string; timestamps and binary via tags |
| **Reuse** | Anchors (`&`), aliases (`*`), merge (`<<`) |
| **Advanced** | Tags (`!!`), directives (`%YAML`, `%TAG`), complex keys, `!!set`/`!!omap` |
| **Portability** | 1.1 vs 1.2 differences; quote `yes/no/on/off`, octal, versions, ports |
| **Safety** | `safe_load`, alias-bomb limits, no secrets in plaintext |
| **Practice** | Lint in CI; validate with the real consumer; 2-space indentation; `---` between Kubernetes resources |

> **One mental model:**
> YAML = **indentation-driven tree** of three node types — **mappings** (`key: value`), **sequences** (`- item`), and **scalars** (everything else). Tags declare types, anchors reuse nodes, and quoting removes ambiguity.

Every concept above appears as a working example in [`examples.yml`](examples.yml).
