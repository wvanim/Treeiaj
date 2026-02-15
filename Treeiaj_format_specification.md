# TREEIA-JSON – Format Specification (version 1.0)

## 1. Introduction

**TREEIA-JSON** (Determined Tree Structure, JSON representation) is a text format designed to represent trees of arrays and objects in a deterministic way.

Treeia defines the syntax for data organization.
To understand the scope of Treeia, imagine a schema that represents a tree — not to describe its content, but to define its shape.

* the **definitions** (structures, strings, colors) stored in **libraries**,
* and the **script** that instantiates these definitions.

The format is entirely determined by the JSON document and its libraries, with no heuristics or contextual interpretation.

This document describes **version 1.0** of the TREEIA-JSON format.

---

## 2. General principles

* The file is a valid JSON object (RFC 8259).
* All strings are **UTF-8** (by nature of JSON).
* Property names and values are strictly defined below.
* The format is **deterministic**: two documents representing the same information are identical (modulo object property order, which has no semantic meaning).
* No JSON comments are allowed in the data stream (although tools may tolerate them during editing, they are not part of the exchange format).
* **Extensibility**: additional properties may be added in the sections provided for this purpose (`declarations`), but must not modify the interpretation of standard properties.

---

## 3. Structure of a TREEIA-JSON document

A TREEIA-JSON document is a **root object** containing the following properties, **all optional except `script`**:

```json
{
  "header":       { ... },   // optional, version information
  "declarations": { ... },   // optional, reserved for extensions
  "strings":      [ ... ],   // optional, string library
  "colors":       [ ... ],   // optional, color library
  "structs":      [ ... ],   // optional, structure library
  "script":       [ ... ]    // mandatory, instruction sequence
}
```

**Rule**: Property order is not significant.

---

## 4. HEADER (object, optional)

| Property     | Type      | Description                                    |
| ------------ | --------- | ---------------------------------------------- |
| `magic`      | string    | **Required if present**; must be `"TREE_DET"`. |
| `version`    | [int,int] | **Required if present**; e.g. `[1,0]`.         |
| `flags`      | integer   | Reserved, must be `0` in v1.0.                 |
| `extensions` | object    | Reserved for future extensions.                |

**Example**:

```json
"header": {
  "magic": "TREE_DET",
  "version": [1, 0],
  "flags": 0
}
```

If the header is absent, the default version is `[1,0]` and flags are `0`.

---

## 5. DECLARATIONS section (object, optional, reserved)

This section is intended for future extensions (global declarations, parameters, constants, etc.).
In version 1.0, if present, it MUST be an empty object (`{}`).
No other format is defined at this time.

---

## 6. STRINGS library (array of strings)

The `strings` library is a **JSON array** containing UTF-8 strings, **deduplicated** (the same string appears only once).

**Example**:

```json
"strings": [
  "pixel",
  "rem",
  "Documentation of the coord structure"
]
```

**Access**: the index of the string in the array serves as the reference (integer, base 0).
References to a string (`string_ref`) in other sections are expressed by this index.

---

## 7. COLORS library (array of colors)

The `colors` library is a **JSON array** whose elements each represent an RGBA color.
Two representations are allowed, at the encoder’s choice:

* **Array of 4 integers**: `[R, G, B, A]` (each component from 0 to 255).
* **Hexadecimal string**: `"#RRGGBBAA"` (8 characters after `#`).

Access is by index (integer, base 0).
Deduplication is recommended but not mandatory.

**Examples**:

```json
"colors": [
  [255, 0, 0, 255],
  "#00FF00FF"
]
```

---

## 8. STRUCTS library (array of structures)

The `structs` library is a **JSON array** containing structure declarations.
Each structure is a JSON object with the following properties:

| Property  | Type            | Description                                                |
| --------- | --------------- | ---------------------------------------------------------- |
| `id`      | integer         | Unique identifier (used in the script). Must be >= 0.      |
| `name`    | string          | Structure name.                                            |
| `doc`     | integer or null | Index in `strings` for documentation, or `null` if absent. |
| `version` | integer         | Declaration version (0 if unused).                         |
| `flags`   | integer         | Reserved, must be 0 in v1.0.                               |
| `params`  | array           | Parameter array (see §8.1).                                |

### 8.1 Parameter format

Each parameter is represented by a **JSON array** of variable length, according to the following rules:

| Position | Name          | Type                | Mandatory                     |
| -------- | ------------- | ------------------- | ----------------------------- |
| 0        | `name`        | string              | always                        |
| 1        | `type`        | string or `"union"` | always                        |
| 2        | `optional`    | boolean             | always                        |
| 3        | `union_types` | array[string]       | **only if `type == "union"`** |

**Rules**:

* If `type` is not `"union"`, the array has **3 elements**.
* If `type == "union"`, the array has **4 elements**; the fourth is an array of strings listing allowed types (e.g. `["float", "const_predef"]`).

**Recognized types** (correspond to opcodes in the binary format):

`"boolean"`, `"uint8"`, `"uint16"`, `"int16"`, `"int32"`, `"float"`, `"word"`, `"string_ref"`, `"post_typed"`, `"color_rgba"`, `"color_ref"`, `"const_predef"`.

`post_typed` is a composite type representing a pair `[number, const_predef]`.
It can be used as a parameter type or inside `union_types`.

**Examples**:

```json
["left_value", "float", false]
["left_unit", "union", false, ["float", "const_predef"]]
["optional_flag", "boolean", true]
```

**Note**: this compact representation is mandatory in the exchange format. An expanded version (named object) may be used internally for readability, but must be normalized during serialization.

---

## 9. SCRIPT (instruction array)

`script` is a **JSON array** of sequentially interpreted instructions.
Each instruction is itself an array or an object depending on its type.

### 9.1 `struct_instance` instruction

Represents the instantiation of a previously declared structure.

**Compact format** (recommended):

```json
["instance", <struct_name>, [value1, value2, ...]]
```

* `struct_name`: word (reference to a structure name in `structs`).
* The third element is an **array** containing parameter values, **in the order of the structure definition**.

**Explicit format** (alternative):

```json
{"type": "instance", "struct": <struct_name>, "values": [...]}
```

Values are expressed directly in JSON according to their type:

* `boolean`: `true` / `false`
* `uint8`, `uint16`, `int16`, `int32`: `number` (within type limits)
* `float`: `number` (IEEE 64-bit, native JSON)
* `word`: `string`
* `string_ref`: `integer` (index in `strings`)
* `post_typed`: `[number, string]`
  // example `[10,"#px"]`, `[5,"#s"]`, `[120,"#%"]`, `[90,"#deg"]`
* `color_rgba`: `[R,G,B,A]` or `"#RRGGBBAA"`
* `color_ref`: `integer` (index in `colors`)
* `const_predef`: **special string** corresponding to the constant (e.g. `"#px"`).

**Parameter serialization rules** (identical to binary):

* all optional parameters must be at the end of the list.
* If a parameter is **mandatory** and of a **single type** → its value is placed **directly** in the array (no type indicator).
* If a parameter is **mandatory** and of **union type** → the value is preceded by its **type** (as a string) in an array `[type, value]`.
* If a parameter is **optional** and **absent** → nothing is written (the values array has one fewer element).
* If a parameter is **optional** and **present** → the value is preceded by its type, as for union.

**Example with `coord`** (structure id=1, parameters in order):

```json
["instance", 1, [
  10.5,   // left_value : float, mandatory, single type → direct value
  "#px",  // left_unit  : const_predef, mandatory, single type → direct value
  20.0,   // top_value
  "#rem"
]]
```

**Example with union and optional** (structure id=2, parameters: `[union float/const, optional boolean]`):

```json
["instance", 2, [
  ["float", 3.14],
  ["", true
]]
```

---

### 9.2 Predefined constants (`const_predef`)

Constants are represented by **special strings** starting with `#`.

| Constant | JSON representation |
| -------- | ------------------- |
| `"px"`   | `"#px"`             |
| `"%"`    | `"#%"`              |
| `"rem"`  | `"#rem"`            |

These strings are used **directly** as values in the script when the expected type is `const_predef`.
They are also used in the `type` field of a parameter or in `union_types`.

**Extension**: new constants may be defined in a future version; they will follow the same `#name` convention.

---

### 9.3 Other instructions (reserved)

The following instructions are not used in v1.0 but are reserved for extensions:

* `"array"` (heterogeneous array) – could be represented directly by a JSON array.
* `"color_rgba"` – may be used as inline value.
* `"word"` – an ordinary JSON string.
* etc.

**Recommendation**: for atomic values (booleans, numbers, strings), use the natural JSON representation. No opcode is required.

---

## 10. Complete example (`coord.treeia.json` file)

```json
{
  "header": {
    "magic": "TREE_DET",
    "version": [1, 0],
    "flags": 0
  },
  "structs": [
    {
      "id": 1,
      "name": "coord",
      "doc": null,
      "version": 0,
      "flags": 0,
      "params": [
        ["left_value", "float", false],
        ["left_unit", "const_predef", false],
        ["top_value", "float", false],
        ["top_unit", "const_predef", false]
      ]
    }
  ],
  "script": [
    ["instance", 1, [10.5, "#px", 20.0, "#rem"]]
  ]
}
```

This document is **bijective** with the binary example provided in the original TREEIA specification.

---

## 11. Implementation notes

### 11.1 Deduplication

* `strings`: **mandatory** deduplication (the same string must appear only once).
* `colors`: deduplication recommended but not required.
* `structs`: no deduplication imposed (`id`s are unique by definition).

---

### 11.2 Importing external modules

When a structure declaration is imported from another TREEIA-JSON file:

1. Extract all references (`string_ref`, `color_ref`) present in the declaration.
2. Resolve these references in the source module libraries (via indexes).
3. Add the corresponding strings and colors to the host file libraries (with deduplication).
4. Update indexes in the imported declaration.
5. Add the declaration (with its `id` possibly reassigned) to the host `structs` array.

This operation is performed **by the build tool** (Ntreeia), not by the format itself.

---

### 11.3 Error handling

* Any document that does not comply with the described schema (types, required properties, etc.) is invalid.
* References (`struct_name`, indexes in `strings`, `colors`) must point to existing entries, otherwise the document is invalid.
* Instance value arrays must contain exactly the number of elements corresponding to the **present** parameters of the structure, after applying optional omission rules.

---

## 12. Summary of differences with binary TREEIA format

| Element                  | Binary TREEIA format                | TREEIA-JSON format                               |
| ------------------------ | ----------------------------------- | ------------------------------------------------ |
| **Physical medium**      | Binary file, little-endian, offsets | UTF-8 text file, JSON structure                  |
| **Header**               | Fixed 32 bytes, offsets             | Optional `header` object, metadata               |
| **Libraries**            | Sections with offset tables         | JSON arrays (`strings`, `colors`, `structs`)     |
| **Strings**              | `uint16 length` + UTF-8             | JSON string                                      |
| **Colors**               | 4 bytes R,G,B,A                     | `[R,G,B,A]` or `"#RRGGBBAA"`                     |
| **Parameters**           | Flags + opcodes + word              | Compact array of 3 or 4 elements                 |
| **Script**               | Opcode stream                       | JSON instruction array                           |
| **Predefined constants** | Opcode `0x10` + 2/3-byte payload    | Special string `"#px"`, `"#%"`, `"#rem"`         |
| **Types**                | Numeric opcodes                     | Literal strings (`"float"`, `"const_predef"`, …) |
| **Instances**            | struct_name + raw data              | `["instance", name, {...}]`                      |

---

## 13. Conclusion

It preserves the philosophy of strict separation between definitions and instances, strong typing, determinism, and offers **natural extensibility** thanks to JSON’s flexibility.

This document serves as the **single reference** for implementers of parsers, generators, and structured data exchange tools following the TREEIA model.

---

*Document written on February 12, 2026, adapted from the binary STS specification version 1.0.*

---
