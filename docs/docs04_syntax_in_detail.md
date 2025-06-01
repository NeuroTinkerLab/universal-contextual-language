# UCL Syntax in Detail

This document provides a more detailed explanation of the textual syntax for UCL (Universal Contextual Language) Version 4.2 "Enhanced," as defined in the [main specification](../SPECIFICATION.MD). It aims to clarify how each part of a UCL message is constructed.

Refer to Part 1.4 of the specification for the guiding ABNF grammar. `CANONICAL_SPACE` in the ABNF implies exactly one space character (`%x20`) in the Canonical Form. Parsers in Permissive Mode may tolerate more whitespace.

## 1. Overall Message Structure

A UCL message is a single UTF-8 text string composed of several distinct parts, many of which are optional.

**Full Structure:**
`[PrefixDeclarations] [SourcePart CANONICAL_SPACE] [Direction CANONICAL_SPACE] TargetPart CANONICAL_SPACE OperationPart [CANONICAL_SPACE ModifiersPart] [CANONICAL_SPACE PayloadSeparator CANONICAL_SPACE PayloadPart] CANONICAL_SPACE ContextSeparator CANONICAL_SPACE ContextStackPart`

Let's examine each component.

## 2. Prefix Declarations (`PrefixDeclarations`)

*   **Purpose:** To define short aliases (prefixes) for long base URIs, making UCL-IDs more concise.
*   **Syntax:** `@prefix declared_prefix: <URI_Base>`
    *   Must appear at the very beginning of the UCL message, one declaration per line.
    *   Each declaration is followed by a `NEWLINE` (`%x0A`).
*   **Example:**
    ```ucl
    @prefix schema: <http://schema.org/>
    @prefix myapp: <http://example.com/myapp-ontology#>
    // Rest of the UCL message follows...
    ```

## 3. Source Part (`SourcePart`)

*   **Purpose:** Identifies the originator of the message.
*   **Syntax:** A `UCLID` (see [UCL-IDs and Prefixes](./07_ucl_ids_and_prefixes.md) for details on UCL-ID format).
*   **Optionality:** Optional.
*   **Example:** `ucl:id:UserX123`

## 4. Direction (`Direction`)

*   **Purpose:** Indicates the primary flow of the message.
*   **Syntax:** One of the following characters:
    *   `>` (Source to Target; default if omitted and not inferable)
    *   `<` (Target to Source; e.g., for a response)
    *   `<>` (Bidirectional or broadcast)
*   **Optionality:** Optional.
*   **Example:** `>`

## 5. Target Part (`TargetPart`)

*   **Purpose:** Identifies the primary recipient or the main subject/scope of the message.
*   **Syntax:** A `UCLID`.
*   **Optionality:** Mandatory.
*   **Example:** `ucl:service:WeatherService`

## 6. Operation Part (`OperationPart`)

*   **Purpose:** Defines the main action or intent of the message.
*   **Syntax:** `OperationVerb CANONICAL_SPACE UCLID_OperationSpecific`
    *   **`OperationVerb`:** A textual verb. Standard verbs include:
        *   `read`
        *   `execute`
        *   `query`
        *   `create`
        *   `delete`
        *   `subscribe`
        *   `notify`
        *   Alternatively, a custom verb can be a `UCLID` (e.g., `myapp:verb:customProcess`).
    *   **`UCLID_OperationSpecific`:** A `UCLID` further specifying the operation or the primary type/concept being acted upon.
*   **Optionality:** Mandatory.
*   **Examples:**
    *   `query schema:Book`
    *   `execute ucl:action:sendEmail`
    *   `create myapp:type:UserAccount`

## 7. Modifiers Part (`ModifiersPart`)

*   **Purpose:** Adds optional qualifiers that alter the behavior of the operation.
*   **Syntax:** One or more `^UCLID` tokens, separated by `CANONICAL_SPACE`.
    *   `^` is the modifier prefix.
*   **Optionality:** Optional.
*   **Example:** `^ucl:mod:async ^ucl:mod:highPriority`

## 8. Payload Separator (`PayloadSeparator`)

*   **Purpose:** Separates the operation and modifiers from the payload.
*   **Syntax:** `:` (colon character)
*   **Optionality:** Mandatory if a `PayloadPart` is present; otherwise, it must be omitted.

## 9. Payload Part (`PayloadPart`)

*   **Purpose:** Contains the data or parameters for the operation.
*   **Syntax:** A single `Value`. A `Value` can be:
    *   A `UCLID` (a reference).
    *   A `LiteralValue` (String, Number, Boolean, Null, DateTime, Duration, Binary).
    *   A `ListValue` (`[ Value1, Value2, ... ]`).
    *   A `MapValue` (`{ "Key1": Value1, "Key2": Value2, ... }`).
    *   Lists and Maps can be nested.
*   **Optionality:** Optional (if an operation requires no data).
*   **Examples:**
    *   Single UCL-ID: `wd:Q42`
    *   String Literal: `"This is a payload."`
    *   Number Literal: `123.45`
    *   List: `[ "apple", "banana", "cherry" ]`
    *   Map: `{ schema:name: "John Doe", ucl:param:age: 30 }`
    *   Nested: `{ ucl:param:user: { schema:givenName: "Jane", schema:familyName: "Doe" }, ucl:param:items: [1, 2, 3] }`
    *   (See [Payload Data Types](./05_payload_data_types.md) for full details on literals, lists, and maps).

## 10. Context Separator (`ContextSeparator`)

*   **Purpose:** Separates the main message (up to payload) from the context stack.
*   **Syntax:** `#` (hash character)
*   **Optionality:** Mandatory.

## 11. Context Stack Part (`ContextStackPart`)

*   **Purpose:** Provides the interpretive frame for the message.
*   **Syntax:** One or more `UCLID`s, separated by `/` (slash character).
    *   Order is significant: most specific context on the left, most general on the right.
*   **Optionality:** Mandatory (at least one context must be present, e.g., `ucl:context:Generic`).
*   **Example:** `ucl:context:FinancialTransaction / schema:Invoice / ucl:standard:UBL`

## Whitespace Handling

*   **Canonical Form:** Exactly one space (`CANONICAL_SPACE`) is used between top-level message components as defined in the ABNF. Inside `ListValue` and `MapValue`, the canonical form for separators (`,` and `:`) might also imply specific spacing (e.g., `[value1, value2]` and `{key1:value1, key2:value2}`). This requires precise definition if absolute canonical textual output is needed.
*   **Permissive Mode:** A parser in permissive mode may tolerate multiple spaces between components and around internal separators (like `,` in lists or `:` in maps), and should normalize these to the canonical representation. Leading/trailing whitespace in the overall message should also be trimmed.

Understanding these syntactic components is the first step to constructing and interpreting valid UCL messages. The semantic meaning of these components, especially UCL-IDs, is derived from their definitions in associated vocabularies or ontologies.

---

Next, we'll dive deeper into:
*   [Payload Data Types in Depth](./05_payload_data_types.md)