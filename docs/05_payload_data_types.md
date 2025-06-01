# UCL Payload Data Types in Depth

The `PayloadPart` of a UCL message is where the primary data for an operation or the result of a query resides. It consists of a single UCL `Value`. This document details the various forms a `Value` can take, as specified in Part 3 of the [UCL Specification](../SPECIFICATION.MD).

A `Value` in UCL can be one of the following:
1.  A **UCL-ID** (acting as a reference or an enumerated value)
2.  A **Literal Value** (String, Number, Boolean, Null, DateTime, Duration, Binary)
3.  A **List Value** (an ordered collection of Values)
4.  A **Map Value** (an unordered collection of key-Value pairs)

## 1. UCL-ID as a Value

A UCL-ID (a URI or a CURIE like `schema:Book` or `wd:Q42`) can be directly used as a value within a payload.

*   **Purpose:**
    *   To reference a specific entity or concept (e.g., identifying a user, a product, or a standard).
    *   To represent an enumerated value from a controlled vocabulary (e.g., `ucl:value:Ascending` for sorting direction).
*   **Example:**
    ```ucl
    // ... OperationPart ... :
    { 
      schema:author: wd:Q3395, // wd:Q3395 (George Orwell) is a UCL-ID value
      ucl:param:status: ucl:status:Completed // ucl:status:Completed is a UCL-ID value
    }
    // ... ContextStackPart ...
    ```

## 2. Literal Values (`LiteralValue`)

These are concrete data values.

### 2.1. Null (`ucl:val:null`)

*   **Representation:** The specific UCL-ID `ucl:val:null`.
*   **Purpose:** Represents the intentional absence of a value, a non-applicable field, or an undefined state. It is distinct from an empty string or zero.
*   **Example:** `{ ucl:param:middleName: ucl:val:null }`

### 2.2. Boolean

*   **Representation:** The lowercase literals `true` or `false`.
*   **Purpose:** Represents logical truth values.
*   **Example:** `{ ucl:param:isActive: true }`

### 2.3. String

*   **Representation:** A sequence of UTF-8 characters enclosed in double quotes (`"`).
*   **Escaping:** `\` is used to escape double quotes (`\"`), backslashes (`\\`), and control characters (`\n`, `\r`, `\t`, `\b`, `\f`). Unicode characters can be represented as `\uXXXX`.
*   **Purpose:** Represents textual data.
*   **Example:** `{ schema:name: "The Hitchhiker's Guide to the Galaxy" }`

### 2.4. Number

*   **Representation:** A numeric literal, which can be an integer (e.g., `42`, `-1`), a decimal (e.g., `3.14`, `-0.05`), or use scientific E-notation (e.g., `6.022e23`).
*   **Interpretation:** Parsers should support IEEE 754 double-precision for floating-point numbers. For arbitrary-precision integers, representation as a string qualified by type/context or a dedicated `ucl:datatype:bigint("...")` is recommended (see specification Part 3.2.4).
*   **Purpose:** Represents numerical data.
*   **Example:** `{ schema:height: 1.88, ucl:param:count: 100 }`

### 2.5. DateTime

*   **Representation:** An ISO 8601 formatted string literal, **including a timezone designator** (preferably `Z` for UTC).
*   **Purpose:** Represents a specific point in time.
*   **Example:** `{ schema:datePublished: "2024-03-15T12:30:00Z" }`

### 2.6. Duration

*   **Representation:** An ISO 8601 duration formatted string literal.
*   **Purpose:** Represents a length of time.
*   **Example:** `{ schema:cookTime: "PT45M" }` (45 minutes)

### 2.7. Binary

*   **Representation (Textual UCL):** `ucl:datatype:binary("BASE64_ENCODED_STRING")`
    *   The `BASE64_ENCODED_STRING` is the Base64 (RFC 4648) representation of the binary data.
*   **Purpose:** Represents arbitrary binary data (e.g., images, audio files, serialized objects not otherwise representable in UCL).
*   **Note:** For large binary objects, referencing an external resource via a UCL-ID (URI) is often preferred over embedding large Base64 strings. The UCL-Bin format will offer more efficient native binary representation.
*   **Example:** `{ ucl:param:avatarImage: ucl:datatype:binary("iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=") }` (a 1x1 red pixel PNG)

## 3. List Value (`ListValue`)

*   **Representation:** An ordered sequence of zero or more `Value`s, enclosed in square brackets `[...]` and separated by commas `,`.
*   **Purpose:** Represents ordered collections, arrays, or sequences.
*   **Nesting:** List elements can be any `Value` type, including other Lists or Maps.
*   **Empty List:** `[]`
*   **Examples:**
    *   `{ ucl:param:tags: [ "sci-fi", "comedy", "adventure" ] }`
    *   `{ ucl:param:chapters: [ { schema:name: "Chapter 1", ucl:param:pages: 20 }, { schema:name: "Chapter 2", ucl:param:pages: 25 } ] }` (List of Maps)
    *   `{ ucl:param:matrix: [ [1,0,0], [0,1,0], [0,0,1] ] }` (List of Lists)

## 4. Map Value (`MapValue`)

*   **Representation:** An unordered collection of zero or more key-value pairs, enclosed in curly braces `{...}`. Pairs are separated by commas `,`, and keys are separated from values by a colon `:`.
*   **Keys:** Can be a `UCLID` (semantically preferred, e.g., `schema:name`) or a `StringLiteral` (e.g., `"custom_field"`).
*   **Values:** Can be any `Value` type, allowing for nested Maps and Lists.
*   **Purpose:** Represents objects, dictionaries, associative arrays, or structured records.
*   **Empty Map:** `{}`
*   **Duplicate Key Policy:** In Strict Mode, duplicate keys are an error. In Permissive Mode, the last occurrence of a key typically overrides previous ones, and a warning should be issued.
*   **Examples:**
    ```ucl
    { 
      schema:name: "John Doe",
      ucl:param:age: 30,
      ucl:param:contact: {
        schema:email: "john.doe@example.com",
        schema:telephone: "+1-555-0100"
      },
      ucl:param:roles: [ ucl:role:Admin, ucl:role:Editor ]
    }
    ```

Understanding these payload data types is essential for constructing UCL messages that carry meaningful and well-structured information for AI systems and other machine processors. The ability to nest Lists and Maps provides the flexibility to represent complex, hierarchical data.

---

Next, we'll explore:
*   [Understanding the Context Stack](./06_context_stack.md)