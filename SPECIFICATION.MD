# UCL (Universal Contextual Language) Specification - Version 4.2 "Enhanced"

## Part 1: Introduction, Scope, Principles, and General Message Structure

### 1.1. Introduction

UCL (Universal Contextual Language) is a formal language designed for the representation of structured, contextualized, and actionable information. Its purpose is to facilitate unambiguous communication and semantic interoperability between heterogeneous software systems, including Large Language Models (LLMs) and AI agents. This specification defines version 4.2 of UCL, named "Enhanced," which incorporates improvements for efficiency, readability, and integration with modern AI architectures.

### 1.2. Scope and Core Principles

*   **Primary Scope:** To provide a universal, efficient, and semantically rich format that enables:
    *   The precise description of data, requests, commands, and configurations.
    *   Unambiguous interpretation by machine systems.
    *   Seamless integration with the interaction paradigms of LLMs.
    *   Interoperability between systems based on shared vocabularies and open standards.

*   **Guiding Principles:**
    1.  **Semantic Unambiguity:** Every UCL construct, when interpreted within its context stack, must have a unique meaning.
    2.  **Rigorous Machine Processability:** The syntax must be formally defined to allow for deterministic parsing and reliable validation.
    3.  **Facilitated Human Readability:** While prioritizing machine processability, UCL's textual form aims for sufficient readability to enable human development, debugging, and understanding.
    4.  **Explicit Contextualization:** The meaning and interpretation of a UCL message are intrinsically tied to its `ContextStackPart`.
    5.  **Semantic Interoperability:** Promote the use of globally unique identifiers (URIs) and tying into standard or shared vocabularies and ontologies.
    6.  **Controlled Extensibility:** The language must allow for the introduction of new concepts, actions, and contexts in a standardized and manageable way.
    7.  **Representation Efficiency:** Balance semantic expressiveness with compactness, especially in its binary serialization form.

### 1.3. Canonical Form vs. Permissive/Expanded Form

To balance machine processability with human readability/practicality, UCL distinguishes two textual forms:

*   **Canonical Form:** This is the strict, normalized textual representation of a UCL message. It serves as the reference for conformance validation and as the expected output of normalization processes. Features:
    *   Uses exactly one single space character (`%x20`) as a separator between major message components and where specified by the grammar.
    *   Contains no comments.
    *   UCL-IDs in CURIE-like format (`prefix:local_part`) must use prefixes declared via `@prefix` at the beginning of the message or prefixes predefined by the specification. Full URIs are generally not preferred in canonical form if a prefix is declarable/available (this rule prioritizes brevity for the canonical form, but parsers must still handle full URIs).
    *   Uses double quotes (`"`) for strings and standard escaping conventions.
*   **Permissive/Expanded Form:** This form is tolerated by a UCL parser operating in "permissive" mode. It is intended to ease manual authoring or generation by less precise LLMs.
    *   May admit multiple spaces between components (which will be normalized to a single space).
    *   Could (in future extensions) support comments (e.g., `// comment to end of line`).
    *   May tolerate the use of single quotes (`'`) for strings, normalizing them to double quotes.
    A parser in permissive mode **must** be able to normalize a valid input in Permissive Form to its corresponding Canonical Form. Machine-to-machine communication should ideally use the Canonical Form or the Binary Serialization.

### 1.4. Basic Structure of a UCL Message (Canonical Textual Syntax)

The formal grammar of a UCL message is defined using an ABNF-like (Augmented Backus-Naur Form) notation. `CANONICAL_SPACE` represents a single space character (`%x20`).

```abnf
UCLMessage      = [PrefixDeclarations] [SourcePart CANONICAL_SPACE] [Direction CANONICAL_SPACE] TargetPart CANONICAL_SPACE OperationPart [CANONICAL_SPACE ModifiersPart] [CANONICAL_SPACE PayloadSeparator CANONICAL_SPACE PayloadPart] CANONICAL_SPACE ContextSeparator CANONICAL_SPACE ContextStackPart

PrefixDeclarations = *(PrefixDeclaration NEWLINE)
PrefixDeclaration  = "@prefix" CANONICAL_SPACE DeclaredPrefix ":" CANONICAL_SPACE "<" URI_Base ">"
DeclaredPrefix     = 1*(ALPHA / DIGIT / "_" / "-") ; Cannot start or end with "_" or "-"
URI_Base           = <Valid absolute URI string, typically ending with "/" or "#">
NEWLINE            = %x0A ; LF

SourcePart      = UCLID
Direction       = ">" / "<" / "<>"
TargetPart      = UCLID
OperationPart   = OperationVerb CANONICAL_SPACE UCLID_OperationSpecific
OperationVerb   = "read" / "execute" / "query" / "create" / "delete" / "subscribe" / "notify" / CustomVerb_UCLID
CustomVerb_UCLID= UCLID
UCLID_OperationSpecific = UCLID ; Added for completeness based on usage

ModifiersPart   = 1*(MODIFIER_PREFIX UCLID) *(CANONICAL_SPACE MODIFIER_PREFIX UCLID)
MODIFIER_PREFIX = "^"
PayloadSeparator= ":"
PayloadPart     = Value
ContextSeparator= "#"
ContextStackPart= UCLID *(CONTEXT_STACK_SEPARATOR UCLID)
CONTEXT_STACK_SEPARATOR = "/"

; Further definitions for UCLID, Value, etc., will be in subsequent sections.
```

### 1.5. Minimal Complete UCL Message Example

The following example illustrates a simple request to create an order:

```ucl
@prefix schema: <http://schema.org/>
@prefix sales: <http://example.org/sales-ontology#>

ucl:id:WebAppInstance7 > sales:service:OrderProcessor execute sales:action:createOrder : { schema:identifier:"ORD-2024-001", sales:product:wd:Q12345, sales:quantity:2, sales:totalAmount: { schema:value:199.98, schema:currency:"EUR" } } # sales:context:NewOrderPlacement / ucl:context:RetailTransaction
```

*   **Prefix Declarations:** Define `schema:` and `sales:`.
*   **Source:** `ucl:id:WebAppInstance7`.
*   **Direction:** `>` (implied as `execute` is an action towards the target, but could be explicit).
*   **Target:** `sales:service:OrderProcessor`.
*   **Operation:** `execute sales:action:createOrder`.
*   **Payload:** A map (`{...}`) with order details, including a nested map for the amount.
*   **Context Stack:** `sales:context:NewOrderPlacement / ucl:context:RetailTransaction`.

### 1.6. Relationship Between Textual and Binary Form

UCL is primarily specified in its textual form for clarity and readability. However, for efficient machine-to-machine transmission and processing, a **Canonical Binary Serialization** (conceptually referred to as "UCL-Bin") is defined (see Section 7 for an overview).
The core principle is:
*   Every token and construct of UCL's Canonical Textual Form has a direct and unambiguous mapping to a byte sequence in the Binary Serialization.
*   A binary parser must be able to reconstruct the original message's Canonical Textual Form (or an equivalent AST) without loss of semantic information.
*   The Binary Serialization employs techniques such as mapping frequently used UCL-IDs and strings to integers, variable-length encodings, and dedicated type bytes for maximum compactness and parsing efficiency.

## Part 2: Unique Identifiers (UCL-IDs) and Prefix Declarations

### 2.1. Definition and Purpose of UCL-IDs

A **UCL-ID (Universal Contextual Language Identifier)** is a character string that uniquely identifies a resource. A "resource" in UCL is a broad concept that can include:

*   Concrete or abstract entities (e.g., a person, a product, a service, a philosophical concept).
*   Types or classes of entities (e.g., the type "Book," the type "FinancialTransaction").
*   Properties or attributes of entities (e.g., the property "color," the parameter "quantity").
*   Actions or operations (e.g., the action "createOrder," the verb "findEntities").
*   Contexts (e.g., the context "CreativeWriting," the context "ScientificDataAnalysis").
*   Specific predefined values (e.g., `ucl:val:null`, `ucl:value:AscendingOrder`).

The primary purpose of UCL-IDs is to provide an unambiguous mechanism for referring to these resources, facilitating semantic interoperability and machine processability.

### 2.2. Canonical Format: URI (Uniform Resource Identifier)

*   The canonical and preferred format for a UCL-ID is a valid **URI (Uniform Resource Identifier)** as defined in RFC 3986.
*   The use of **IRIs (Internationalized Resource Identifiers)** as defined in RFC 3987 is recommended to support Unicode characters from diverse languages. IRIs must be converted to their equivalent ASCII URI form for serialization and use in contexts requiring pure URIs (e.g., via the Percent-Encoding process for non-ASCII characters).
*   **Examples of UCL-IDs as URIs:**
    *   `http://schema.org/Book`
    *   `http://www.wikidata.org/entity/Q42` (Douglas Adams)
    *   `http://example.org/my-ontology#OrderIdentifier`
    *   `urn:iso:std:iso:8601` (for the ISO 8601 standard)

### 2.3. Compact Format: CURIE-like (Prefixed Name)

To improve readability and reduce the verbosity of textual UCL messages, a compact format for UCL-IDs, similar to CURIEs (Compact URI Expressions), is supported.

*   **Syntax:** `prefix:local_part`
    *   `prefix`: An alphanumeric prefix name (may contain `_` or `-` but not at the start or end), which must not be empty.
    *   `:`: The colon character, separating the prefix from the local part.
    *   `local_part`: A string of characters representing the local part of the identifier within the namespace defined by the prefix. Permitted characters in the `local_part` should follow guidelines for the path, query, or fragment components of URIs (e.g., excluding spaces, `#`, `?`, `/` unless specifically allowed by the namespace and appropriately escaped if needed in its full URI form).
*   **Resolution:** Every `prefix` used in a UCL message must be mappable to a base URI. The full UCL-ID (URI) is reconstructed by concatenating the prefix's base URI with the `local_part`.
*   **Examples of UCL-IDs in compact format:**
    *   `schema:Book`
    *   `wd:Q42`
    *   `myont:OrderIdentifier`

### 2.4. Prefix Declarations (`@prefix`)

To enable the use of the `prefix:local_part` compact format, prefixes must be declared.

*   **Declaration Syntax (from Section 1.4):**
    ```abnf
    PrefixDeclaration  = "@prefix" CANONICAL_SPACE DeclaredPrefix ":" CANONICAL_SPACE "<" URI_Base ">"
    DeclaredPrefix     = 1*(ALPHA / DIGIT / "_" / "-") ; Cannot start or end with "_" or "-"
    URI_Base           = <String representing a valid absolute URI, typically ending with a character allowing direct concatenation with local_part, usually "/" or "#">
    ```
*   **Placement:** `@prefix` declarations must appear at the beginning of a UCL message (before any other components like `SourcePart` or `TargetPart`), one per line, followed by a `NEWLINE`.
*   **Scope:** Prefix declarations are local to the UCL message in which they appear.
*   **Example Declarations:**
    ```ucl
    @prefix schema: <http://schema.org/>
    @prefix wd: <http://www.wikidata.org/entity/>
    @prefix myont: <http://example.org/my-ontology#>
    ```
*   **Predefined (Built-in) Prefixes:**
    To reduce verbosity and promote the use of common vocabularies, UCL 4.2 predefines the following prefixes. These prefixes **do not need** to be declared with `@prefix` in a message and can be used directly:
    *   `ucl:` -> `<http://ucl-spec.org/4.2/core#>` (Namespace for UCL core terms)
    *   `rdf:` -> `<http://www.w3.org/1999/02/22-rdf-syntax-ns#>`
    *   `rdfs:` -> `<http://www.w3.org/2000/01/rdf-schema#>`
    *   `xsd:` -> `<http://www.w3.org/2001/XMLSchema#>` (For XML Schema datatypes)
    *   `owl:` -> `<http://www.w3.org/2002/07/owl#>`
    *   `schema:` -> `<http://schema.org/>` (As a common convention, though local declaration is not an error)
    *   `wd:` -> `<http://www.wikidata.org/entity/>` (As a common convention)
    UCL parsers **must** recognize these predefined prefixes. For other common vocabularies (e.g., `dcterms:`, `foaf:`, `skos:`), explicit `@prefix` declaration is recommended for maximum clarity, although a parser *may* be configured with additional "well-known" prefixes.

### 2.5. Order of Prefix Resolution and Error Handling

When a UCL parser encounters a UCL-ID in `prefix:local_part` format, it must resolve it to a full URI by following this order of precedence:

1.  **Local Prefix Map:** Search for the `prefix` among those declared with `@prefix` within the current message. If found, concatenate the corresponding `URI_Base` with the `local_part`.
2.  **Predefined Prefix Map:** If not found in the local map, search for the `prefix` among the predefined prefixes of the UCL specification. If found, use the corresponding base URI.
3.  **Configured External Registry/Registries (Optional):** If not resolved locally or as predefined, the parser *may* (if configured to do so) consult one or more external prefix registries.
    *   The specification of how to query such registries (e.g., HTTP endpoint, response format like JSON-LD or simple JSON `{"prefix": "baseURI"}`) is outside the UCL core specification, but systems should aim for de facto standards.
    *   Parsers **must** implement a caching mechanism for resolutions from external registries to improve performance and reduce network dependencies. Cache entries should have an expiration policy (TTL).
4.  **Resolution Error:** If the `prefix` cannot be resolved through any of the above mechanisms, the parser **must** signal an error (e.g., "Unknown Prefix Error: prefix 'xyz' is not declared or recognized"). The UCL message must be considered invalid.

### 2.6. Versioning of UCL-IDs

To manage the evolution of vocabularies and ontologies, UCL-IDs can include version information.

*   **Recommended Convention:** `UCLID_Base@version_string`
    *   `UCLID_Base`: The primary identifier (URI or CURIE).
    *   `@`: A separator character (the "at" sign).
    *   `version_string`: A string representing the version. The use of **Semantic Versioning (SemVer)** (e.g., `1.0.0`, `2.1.3-beta`) is **strongly recommended**.
*   **Interpretation:**
    *   A parser/system receiving a versioned UCL-ID can use this information to select the correct definition or behavior for that specific version of the concept.
    *   The policy for handling versions (e.g., reject if exact version not supported, fallback to nearest compatible version, issue a warning) is the responsibility of the receiving system or may be defined within the UCL-ID's registry/ontology.
    *   If a UCL-ID is used without a version specifier, it is implicitly assumed to refer to the "latest stable" version or a default version as defined by the namespace manager of the UCL-ID.
*   **Example:** `schema:Book@15.0`, `myont:Order@2.1.0`

## Part 3: Literal and Structured Data Types (Payload)

The `PayloadPart` of a UCL message carries the actual data or parameters for an operation. It is defined as a single `Value`. This section defines the `Value` types supported by UCL 4.2.

### 3.1. Definition of `Value`, `SingleValue`, and `LiteralValue` (Grammar Reference)

From the grammar (Section 1.4):
```abnf
Value           = UCLID / LiteralValue / ListValue / MapValue
SingleValue     = UCLID / LiteralValue
LiteralValue    = NullLiteral / BooleanLiteral / StringLiteral / NumberLiteral / DateTimeLiteral / DurationLiteral / BinaryLiteral
```
A `Value` can be a reference to another entity/concept (`UCLID`), a literal value (`LiteralValue`), a list (`ListValue`), or a map (`MapValue`).

### 3.2. Literal Types (`LiteralValue`)

*   **3.2.1. Null (`NullLiteral`)**
    *   **Representation:** The predefined UCL-ID `ucl:val:null`.
    *   **Semantics:** Represents the absence of a value or a non-applicable value.

*   **3.2.2. Boolean (`BooleanLiteral`)**
    *   **Representation:** The textual literals `true` or `false` (exactly these lowercase strings).
    *   **Semantics:** Represents logical truth values.

*   **3.2.3. String (`StringLiteral`)**
    *   **Representation:** A sequence of zero or more Unicode characters enclosed in double quotes (`"`).
    *   **Encoding:** Strings must be interpreted as UTF-8 encoded.
    *   **Escaping:**
        *   Double quote (`"`) and backslash (`\`) characters within the string **must** be escaped with a preceding backslash (`\"` and `\\` respectively).
        *   The following standard escape sequences are supported:
            *   `\n`: Newline (LF)
            *   `\r`: Carriage Return (CR)
            *   `\t`: Horizontal Tab
            *   `\b`: Backspace
            *   `\f`: Form Feed
            *   `\uXXXX`: Unicode character specified by four hexadecimal digits (`XXXX`). For characters outside the Basic Multilingual Plane (BMP), use the UTF-16 surrogate pair representation (two `\uXXXX` sequences).
    *   **Example:** `"This is a \"sample\" string.\nWith a new line."`

*   **3.2.4. Number (`NumberLiteral`)**
    *   **Representation:** A sequence of digits representing a numerical value.
        *   Can be an integer (e.g., `123`, `-45`).
        *   Can be a decimal number, using a period (`.`) as the decimal separator (e.g., `3.14159`, `-0.5`). A decimal number must have at least one digit before or after the period.
        *   May include an optional exponent in scientific notation, indicated by `e` or `E`, followed by an optional sign (`+` or `-`) and one or more digits (e.g., `1.23e-4`, `-5E+10`).
    *   **Numerical Interpretation:**
        *   Parsers **should** support double-precision floating-point numbers according to the IEEE 754 standard.
        *   For integers exceeding the representation capacity of standard 64-bit integer types (or for applications requiring arbitrary precision), it is **recommended** to represent them as a `StringLiteral` and semantically qualify them via the `ContextStackPart` or the type of the parameter to which they are assigned, or by using a dedicated type UCL-ID such as `ucl:datatype:bigint("123...999")`. The binary serialization specification may define a native `bigint` type.
    *   **Examples:** `42`, `-10`, `3.14`, `6.022e23`

*   **3.2.5. DateTime (`DateTimeLiteral`)**
    *   **Representation:** A `StringLiteral` that **must** conform to the complete date-time format of ISO 8601.
    *   **Mandatory Timezone Designator:** The string **must** include a timezone designator:
        *   `Z` for UTC (Coordinated Universal Time) – **Strongly Recommended**.
        *   An explicit offset from UTC (e.g., `+HH:MM` or `-HH:MM`).
    *   The absence of a timezone designator makes the value ambiguous and **should** be treated as an error by a parser in Strict Mode, or interpreted as UTC with a warning in Permissive Mode (though the latter is discouraged for interoperability).
    *   **Examples:** `"2024-03-15T14:30:00Z"`, `"2024-03-15T16:30:00+02:00"`

*   **3.2.6. Duration (`DurationLiteral`)**
    *   **Representation:** A `StringLiteral` that **must** conform to the duration format of ISO 8601.
    *   **Examples:** `"PT10H"` (10 hours), `"P3M10D"` (3 months and 10 days), `"P1Y2M3DT4H5M6S"` (1 year, 2 months, 3 days, 4 hours, 5 minutes, 6 seconds).

*   **3.2.7. Binary (`BinaryLiteral`)**
    *   **Textual Representation:** A construct resembling a function call or a qualified type is used for semantic clarity, with the Base64 string as its argument.
        *   **Recommended Format:** `ucl:datatype:binary("BASE64_ENCODED_STRING")`
        *   `BASE64_ENCODED_STRING`: The binary data string encoded according to the Base64 standard (RFC 4648), without line breaks.
    *   **Size Considerations (for textual form):**
        *   For very large binary data in textual form, the Base64 string can become excessive. In such cases, applications **should** consider:
            *   References to external binary resources via a `UCLID` (URI).
            *   Multi-part transmission mechanisms at the application protocol level (outside the UCL core textual specification).
        *   The UCL Binary Serialization specification (Section 7) will address the efficient representation of large binary data, potentially including chunking or direct binary data types.
    *   **Optional Attributes (Extensibility):** An application *may* define UCL-IDs for additional parameters to describe binary data if needed (e.g., `ucl:param:originalFileName`, `ucl:param:mediaType`, `ucl:param:compressionAlgorithm`), typically within a `MapValue` containing the `BinaryLiteral`.
    *   **Example:** `ucl:datatype:binary("SGVsbG8gd29ybGQh")` (for "Hello world!")

### 3.3. UCL-ID Reference as a Value

A `UCLID` (as defined in Section 2) can be used directly as a `Value` (and thus as a `SingleValue`).
*   **Semantics:** Represents a reference to the entity, concept, or resource identified by that UCL-ID.
*   **Example:** In a payload `{ ucl:param:author: wd:Q3395 }`, `wd:Q3395` is a `UCLID` used as a value.

### 3.4. Compound Structures

*   **3.4.1. List (`ListValue`)**
    *   **Syntax:** `[` `[` `Value` `*(LIST_SEPARATOR Value)` `]` `]` (Outer square brackets are delimiters; inner indicates optionality of the value list if the list is empty).
        *   Simplified form: `[` `Value_1` `,` `Value_2` `,` ... `,` `Value_N` `]`
        *   `LIST_SEPARATOR` is `,` (comma). In canonical form: `,` (comma with a single space after, unless it's the last element).
    *   **Semantics:** Represents an ordered sequence of zero or more `Value`s.
    *   **Nesting:** Since a `Value` can itself be a `ListValue` or a `MapValue`, lists can contain nested structures.
    *   **Empty List:** Represented as `[]`.
    *   **Example:** `[ "apple", 123, true, { ucl:param:color: "red" } ]`

*   **3.4.2. Map (`MapValue`)**
    *   **Syntax:** `{` `[` `MapPair` `*(MAP_SEPARATOR MapPair)` `]` `}`
        *   Simplified form: `{` `Key_1:Value_1` `,` `Key_2:Value_2` `,` ... `,` `Key_N:Value_N` `}`
        *   `MAP_SEPARATOR` is `,` (comma). In canonical form: `,` (comma with a single space after, unless it's the last pair).
    *   **`MapPair` Syntax:** `(UCLID / StringLiteral) PAIR_SEPARATOR Value`
        *   `PAIR_SEPARATOR` is `:` (colon). In canonical form: `:` (colon with no spaces around it in canonical form of map pairs, or a single space after, depending on strict interpretation of "key: value"). Let's assume no spaces for key:value for max compactness.
    *   **Semantics:** Represents an unordered collection of zero or more key-value pairs.
    *   **Keys:**
        *   Each key **must** be either a `UCLID` or a `StringLiteral`.
        *   The use of `UCLID`s as keys is **strongly recommended** for semantic clarity and interoperability (e.g., `ucl:param:name`, `schema:author`).
        *   StringLiteral keys should be used cautiously, preferably for application-specific fields whose names do not have a standard UCL-ID equivalent.
        *   **Duplicate Key Policy:** If a `MapValue` contains duplicate keys (same string or same resolved URI):
            *   **Strict Mode:** The parser **must** report an error and consider the message invalid.
            *   **Permissive Mode:** The parser *may* accept the message, and the value associated with the key will be that of the *last* occurrence of the key encountered during parsing. It **must** still issue a warning.
    *   **Values:** Each value in a `MapPair` can be any `Value` type, allowing for nesting.
    *   **Empty Map:** Represented as `{}`.
    *   **Example:**
        ```ucl
        {schema:name:"The Book Title",schema:author:wd:Q3395,ucl:param:publicationYear:2024,ucl:param:tags:["science_fiction","classic"],"customApp:internalID":"XYZ-789"}
        ```
        (Note: Example shown with canonical spacing for map pairs and list items).

### 3.5. Edge Case Handling (Summary)

*   **Empty List (`[]`):** Is a valid `ListValue`.
*   **Empty Map (`{}`):** Is a valid `MapValue`.
*   **Duplicate Keys in Map:** Error in Strict Mode; last one wins with a warning in Permissive Mode.
*   **`ucl:val:null` Values:** Are permitted as values in lists and as values associated with keys in maps. The absence of a key in a map is semantically different from a key being present with the value `ucl:val:null`.

---

## Part 4: Semantics of Message Components

This section describes the meaning and purpose of each major component of a UCL message, as defined in the general structure (Section 1.4).

### 4.1. `PrefixDeclarations`

*   **Purpose:** To provide local aliases (prefixes) for base URIs, in order to shorten UCL-IDs within the message body and improve readability.
*   **Semantics:** Each `@prefix prefix_name: <URI_BASE>` declaration establishes a mapping valid for the scope of the current UCL message.
*   **Processing:** They must be processed before any other message components to allow for the correct expansion of CURIEs.

### 4.2. `SourcePart`

*   **Syntax:** `UCLID`
*   **Purpose:** Identifies the entity (user, application, service, device) that originated or sent the UCL message.
*   **Semantics:** Provides information about the origin of the message, useful for:
    *   Authentication and Authorization (the receiving system can verify if the source is permitted to perform the operation).
    *   Auditing and Logging (tracking who did what).
    *   Response Routing (if the response should not go directly back to the immediate originator of the connection).
    *   Personalization (the receiving system can tailor behavior based on the source).
*   **Optionality and Security Implications:**
    *   Syntactically optional.
    *   If `SourcePart` is absent, the sender is considered "anonymous" or "implied by the system/communication channel."
    *   In contexts requiring strong authentication and traceability (e.g., financial transactions, high-privilege commands, sensitive data), a receiving service **can and should** require the presence of a valid and verifiable `SourcePart`. Absence in such contexts may lead to message rejection.

### 4.3. `Direction`

*   **Syntax:** `>` / `<` / `<>`
*   **Purpose:** Indicates the primary logical directionality of the message relative to `SourcePart` and `TargetPart`.
*   **Semantics:**
    *   `>`: The message flows from `SourcePart` (if present, otherwise from an implicit sender) towards `TargetPart`. Typical for commands, requests, sending data. **This is the default value if `Direction` is omitted and not otherwise inferable.**
    *   `<`: The message flows from `TargetPart` (acting as the actual source of the response/notification) towards `SourcePart` (or the implicit recipient of the response/notification). Typical for query responses, command results, event notifications.
    *   `<>`: The message has a bidirectional or broadcast/multicast nature. It could be used for synchronization messages, service announcements in a network, or in some publish/subscribe paradigms where a message is both a publication (from a source) and a potential request others can respond to. The use of `<>` should be well-defined by the application protocol or context.
*   **Optionality and Inference:**
    *   Syntactically optional.
    *   A parser or receiving system *may* attempt to infer the `Direction` if omitted, based on the `OperationVerb`:
        *   `read`, `query`, `execute`, `create`, `delete`: Inferred default is `>`.
        *   `notify`: Inferred default is `<` (if `TargetPart` is the notification recipient and `SourcePart` is the event emitter) or `<>` (if it's a broadcast notification).
        *   `subscribe`: `>` for the subscription request, `<` for subsequent notifications.
    *   If `Direction` is explicitly specified, it takes precedence over any inference.

### 4.4. `TargetPart`

*   **Syntax:** `UCLID`
*   **Purpose:** Identifies the primary entity to which the message is addressed or the primary scope of the operation.
*   **Semantics:** It can represent:
    *   A specific service (e.g., `ucl:service:PaymentGateway`).
    *   A physical or virtual device (e.g., `iot:device:ThermostatRoom101`).
    *   A software agent or module (e.g., `lai:agent:DataAnalyzer`).
    *   A group of entities (e.g., `ucl:group:AllSubscribedUsers`).
    *   An abstract concept or data resource being operated upon (e.g., `wd:Q8336` - the book "Treasure Island," if the operation is `read schema:description`).
*   It is mandatory and fundamental for message routing and interpretation.

### 4.5. `OperationPart`

*   **Syntax:** `OperationVerb CANONICAL_SPACE UCLID_OperationSpecific`
*   **`OperationVerb`:** A textual verb defining the general intent or nature of the operation.
    *   **Standard Recommended Values:**
        *   `read`: Request the current state or representation of a resource (idempotent, no side-effects on the resource).
        *   `execute`: Perform an action, command, or process that may modify a resource's state or have side-effects.
        *   `query`: Request a dataset or information resulting from a complex search or interrogation (typically idempotent).
        *   `create`: Create a new resource.
        *   `delete`: Remove an existing resource.
        *   `subscribe`: Request to receive future notifications related to a resource or event.
        *   `notify`: Send an asynchronous notification regarding an event or state change.
    *   **`CustomVerb_UCLID`:** A `UCLID` can be used instead of a standard verb to define highly domain-specific operations, whose semantics are defined by the ontology to which the UCL-ID belongs.
*   **`UCLID_OperationSpecific`:** A `UCLID` that further qualifies the operation or identifies the primary resource or resource type the operation acts upon.
    *   For `execute`: `ucl:action:startProcess`, `app:verb:sendEmail`.
    *   For `query`: `ucl:verb:findEntities`, `schema:Recipe` (to find recipes).
    *   For `read`: `schema:description` (to read the description of the `TargetPart`).
*   **Overall Semantics:** The entire `OperationPart` defines the main intent of the message. Its precise meaning is determined by the combination of `OperationVerb`, `UCLID_OperationSpecific`, the `PayloadPart`, and the `ContextStackPart`.

### 4.6. `ModifiersPart`

*   **Syntax:** `1*(MODIFIER_PREFIX UCLID) *(CANONICAL_SPACE MODIFIER_PREFIX UCLID)` with `MODIFIER_PREFIX = "^"`
*   **Purpose:** To provide additional qualifiers or flags that modify the behavior of the operation.
*   **Semantics:** Each `^UCLID_Modifier` represents a specific characteristic or request.
    *   **Examples:**
        *   `^ucl:mod:asynchronous`: Request that the operation be performed asynchronously.
        *   `^ucl:mod:dryRun`: Perform a simulation of the operation without applying actual changes.
        *   `^ucl:mod:highPriority`: Indicate high priority for processing.
        *   `^ucl:mod:transactional`: Request that the operation be part of a transaction.
*   The interpretation and support for modifiers depend on the receiving system and the specific operation.

### 4.7. `PayloadSeparator` (`:`)

*   **Purpose:** Separates the "header" part of the message (Source, Direction, Target, Operation, Modifiers) from the `PayloadPart`.
*   **Mandatory:** Present if and only if `PayloadPart` is present.

### 4.8. `PayloadPart`

*   **Syntax:** `Value` (can be a `UCLID`, `LiteralValue`, `ListValue`, or `MapValue`, with support for recursive nesting as defined in Section 3).
*   **Purpose:** Contains the data, parameters, or message body necessary to perform the operation, or the data constituting the result of an operation (if `Direction <`).
*   **Semantics:** Its structure and the meaning of its fields are strictly dependent on the `OperationPart` and are interpreted within the `ContextStackPart`.
    *   For an `execute schema:Recipe` operation with `OperationVerb create`, the payload would contain recipe details (ingredients, instructions) structured as a `MapValue`.
    *   For a `query ucl:verb:findEntities`, the payload might contain filter criteria as a `MapValue`.
    *   For a response (`Direction <`) to a query, the payload would contain the results, perhaps as a `ListValue` of `MapValue`s.

### 4.9. `ContextSeparator` (`#`)

*   **Purpose:** Separates the main body of the message (up to the Payload) from the `ContextStackPart`.
*   **Mandatory:** Always present, even if the `ContextStackPart` contains only a single context.

### 4.10. `ContextStackPart`

*   **Syntax:** `UCLID *(CONTEXT_STACK_SEPARATOR UCLID)` with `CONTEXT_STACK_SEPARATOR = "/"`
*   **Purpose:** Provides the semantic and interpretive frame for the entire UCL message.
*   **Semantics:**
    *   It is an ordered stack of one or more `UCLID`s representing contexts.
    *   The order is significant: **from most specific (left) to most general (right)**. The leftmost context has the highest specificity or precedence in influencing interpretation.
    *   A context can qualify:
        *   The application domain (e.g., `finai:context:StockTrading`, `ucl:context:HealthcareRecord`).
        *   The reference standard or ontology (e.g., `schema:context:UsingSchemaOrgProperties`, `iso:standard:ISO8601_Dates`).
        *   The perspective or role (e.g., `lai:context:AsLegalCounsel`, `ucl:context:AsEndUser`).
        *   The phase of a process (e.g., `pmo:context:DesignPhase`).
    *   The receiving system uses the context stack to:
        *   Disambiguate the meaning of UCL-IDs in the `PayloadPart` or `OperationPart` (e.g., `ucl:param:amount` might mean monetary amount in a financial context, or quantity in an inventory context).
        *   Select appropriate validation or business logic rules.
        *   Tailor behavior or output.
*   **Duplicate UCL-IDs in Stack Policy:**
    *   The presence of the exact same UCL-ID (identical URI and version, if specified) multiple times in the stack is generally redundant. Interpretation should be based on its most specific (leftmost) occurrence.
    *   However, a UCL-ID *could* appear multiple times if each occurrence is qualified by a different version (e.g., `# myapp:context:MyAPI@v1 / myapp:context:MyAPI@v2`), indicating a transition or compatibility between contextual versions. This advanced use must be carefully managed by the system.
    *   Direct cycles (e.g., `# ctx:A / ctx:B / ctx:A` without a version change or qualifying meaning) should be avoided or might be interpreted as an error by the parser if they lead to unresolvable ambiguity.
*   **Minimum Context:** Every UCL message **must** have at least one context. If no specific context is appropriate, a generic context like `ucl:context:Generic` can be used.


## Part 5: Parsing, Interpretation, and Error Handling Rules

This section defines how a UCL 4.2 message should be processed by a parser and an interpretative system, and how errors should be handled.

### 5.1. Parsing Modes

A UCL 4.2 parser **should** support two primary operational modes for handling textual input:

*   **5.1.1. Strict Mode (`strict`)**
    *   **Purpose:** To validate strict conformance to the UCL 4.2 Canonical Form. Ideal for machine-to-machine communication, conformance testing, and when absolute syntactic precision is required.
    *   **Behavior:**
        *   The parser **must** strictly adhere to the ABNF grammar defined for the Canonical Form (Section 1.4 and subsequent sections).
        *   It **must** reject (signal a parsing error for) any deviation from the Canonical Form, including:
            *   Multiple spaces where a single `CANONICAL_SPACE` is required.
            *   Use of single quotes (`'`) for strings instead of double quotes (`"`).
            *   Lack of correct escaping for special characters in strings.
            *   Errors in the structure of lists (`[...]`) or maps (`{...}`).
            *   Unrecognized tokens or tokens in incorrect positions.
            *   Duplicate keys within a `MapValue`.
    *   **Output:** If parsing is successful, it produces an internal representation of the message (e.g., an Abstract Syntax Tree - AST) conforming to the UCL structure. On failure, it produces a detailed error.

*   **5.1.2. Permissive/Relaxed Mode (`permissive`)**
    *   **Purpose:** To facilitate input from less precise sources, such as manual typing or output generated by LLMs that may not perfectly adhere to the canonical syntax.
    *   **Behavior:**
        *   The parser *may* tolerate minor deviations from the Canonical Form and attempt to normalize them.
        *   **Tolerated Deviations (Examples):**
            *   Multiple spaces between components (normalized to a single space).
            *   Leading/trailing spaces in the message or around major delimiters (to be trimmed).
            *   Use of single quotes (`'`) for strings (normalized to double quotes).
            *   (Optional, if defined) Handling of comments.
        *   **Normalization:** If permissive parsing is successful, the parser **should** ideally produce an internal representation equivalent to that which would have been produced from the message's Canonical Form, and *may* also be able to emit the textual Canonical Form.
        *   **Warning Reporting:** For each tolerated and normalized deviation, the parser **should** issue a warning indicating the nature of the deviation and how it was corrected. This helps developers improve the quality of their UCL messages.
        *   **Non-Tolerable Errors:** Even in permissive mode, severe structural errors (e.g., unbalanced brackets, completely unknown tokens in key positions, irrecoverably malformed payload) **must** still result in a parsing error.
        *   **Duplicate Keys in Map:** In permissive mode, the parser *may* accept duplicate keys, with the last occurrence encountered taking precedence, but it **must** issue a warning.

### 5.2. Phases of UCL Message Processing

The complete processing of a UCL message by a receiving system typically involves the following (logical) phases; implementation may vary:

1.  **Phase 1: Syntactic Parsing and Normalization**
    *   **Input:** Text string of the UCL message.
    *   **Process:**
        *   Application of parsing rules (Strict or Permissive) according to the ABNF grammar.
        *   If in Permissive Mode, normalization to Canonical Form (or an equivalent internal representation).
        *   Processing of `@prefix` declarations to build the local prefix map.
        *   Construction of an internal representation of the message, typically an Abstract Syntax Tree (AST) or an equivalent data structure.
    *   **Output:** Syntactically validated UCL message AST (or a parsing error).

2.  **Phase 2: Formal and Contextual Semantic Validation**
    *   **Input:** UCL message AST.
    *   **Process:**
        *   **UCL-ID Resolution:** Expansion of all CURIEs to full URIs using the defined resolution order (local prefixes, predefined, external registries). Signal an error if a prefix or UCL-ID is unresolvable.
        *   **Data Type Validation:** Verify that `LiteralValue`s conform to their types (e.g., a `NumberLiteral` contains a valid number, a `DateTimeLiteral` is valid ISO 8601 with timezone).
        *   **Payload Structure Validation:** Verify that the structure of the `PayloadPart` (if present) is as expected for the specified `OperationPart` within the given `ContextStackPart`. This validation requires knowledge of the operation's and context's semantics.
        *   **Context Validation:** Verify that the UCL-IDs in the `ContextStackPart` are valid and, if the system has contextual coherence logic, that the stack is valid.
        *   **Version Checking (if applicable):** If UCL-IDs include versions, check for compatibility.
    *   **Output:** Enriched and semantically validated AST (or a semantic/contextual error).

3.  **Phase 3: Application-Specific Execution/Interpretation**
    *   **Input:** Enriched and validated AST.
    *   **Process:** This phase is specific to the application or service receiving the UCL message.
        *   The "application engine" or "semantic interpreter" uses the information in the AST (Source, Target, Operation, Payload, Contexts, Modifiers) to perform the requested action.
        *   This might involve: querying a database, calling another service, modifying internal state, generating new content, etc.
        *   Domain-specific and context-specific business logic is applied here.
    *   **Output:** The result of the operation (which could be a new UCL response message with `Direction <`, a side-effect in the system, or no explicit output).

### 5.3. General Error Handling Rules

A system processing UCL **must** implement robust error handling. When an error is detected, the system **should**, where possible, respond with a UCL message describing the error.

*   **UCL Error Message (Example):**
    ```ucl
    @prefix ucl: <http://ucl-spec.org/4.2/core#>
    OriginalMessageTarget < OriginalMessageSource ! ucl:error:processingFailed : { ucl:param:errorCode: ucl:errCode:SyntaxError, ucl:param:errorMessage: "Invalid token '&&' found at position 75.", ucl:param:offendingMessagePart: "...", ucl:param:originalMessageID: "..." } # ucl:context:ErrorResponse / ucl:context:UCLProcessing
    ```

*   **Common Error Categories and Suggested UCL-ID Codes (`ucl:errCode:`):**
    *   **`ucl:errCode:SyntaxError`:** Message does not conform to UCL grammar.
        *   Examples: Unrecognized token, unbalanced brackets/quotes, missing/incorrect delimiters.
    *   **`ucl:errCode:InvalidPrefixDeclaration`:** Error in an `@prefix` declaration.
    *   **`ucl:errCode:UnresolvedPrefix`:** A prefix used in a CURIE was not declared, is not predefined, and could not be resolved via registries.
    *   **`ucl:errCode:UnresolvedIdentifier`:** A full UCL-ID (URI) does not correspond to any known or defined resource in the system or accessible registries.
    *   **`ucl:errCode:InvalidLiteralValue`:** A literal value does not conform to its expected type (e.g., "abc" for a number, a non-ISO 8601 date).
    *   **`ucl:errCode:InvalidPayloadStructure`:** The payload structure is not as expected for the specified operation and context.
    *   **`ucl:errCode:InvalidContext`:** The context stack is invalid or contains incompatible contexts.
    *   **`ucl:errCode:OperationNotSupported`:** The requested `OperationPart` is not supported by the `TargetPart`.
    *   **`ucl:errCode:AuthorizationFailed`:** The `SourcePart` (or implicit sender) is not authorized to perform the operation.
    *   **`ucl:errCode:ResourceNotFound`:** The resource specified in `TargetPart` or `PayloadPart` does not exist.
    *   **`ucl:errCode:InternalServerError`:** A generic server-side error during processing.
*   The error message should provide as much useful information as possible for debugging.

### 5.4. Edge Case Handling (Reference Section 3.5)

The parser must correctly handle edge cases defined for data types:
*   **Empty List (`[]`):** Treated as a valid list with zero elements.
*   **Empty Map (`{}`):** Treated as a valid map with zero key-value pairs.
*   **Duplicate Keys in Map:**
    *   Strict Mode: Error.
    *   Permissive Mode: Last occurrence wins, warning issued.
*   **Absent Payload:** If the syntax allows (i.e., `PayloadSeparator` and `PayloadPart` are omitted) and the operation does not require a payload, this is valid. If an operation requires a payload and it is absent, it is an `ucl:errCode:InvalidPayloadStructure`.


## Part 6: Registries, Extension Mechanisms, and Interoperability

To ensure UCL is a truly universal and interoperable language, mechanisms are needed to manage the definition and discovery of UCL-IDs, as well as to extend the language with new concepts and vocabularies.

### 6.1. External UCL Prefix Registries

*   **Purpose:** To provide a means to resolve UCL-ID prefixes (like `schema:`, `wd:`, `myorg:`) to their corresponding base URIs, especially when these are not declared locally in a message via `@prefix` or are not among the predefined prefixes.
*   **Operation:**
    1.  A system processing UCL *may* be configured with the address(es) of one or more Prefix Registries.
    2.  When encountering a prefix not resolvable locally, the system queries the configured registry/registries.
    3.  Systems **must** implement a caching mechanism for registry responses to minimize network requests and improve performance. Cache entries should have an expiration policy (TTL).
*   **Recommended Registry Protocol and Response Format:**
    *   **Endpoint:** It is recommended that a registry expose an HTTP(S) endpoint (e.g., `https://ucl-registry.example.org/prefixes`).
    *   **Request:** A `GET` request to the endpoint (or a more specific endpoint like `/prefixes/{prefix_name}`) could return the mapping.
    *   **Response Format:** A simple machine-readable format, such as JSON, mapping prefixes to their base URIs is recommended. For richer semantics, JSON-LD could be used.
        *   **Simple JSON Response Example:**
            ```json
            {
              "schema": "http://schema.org/",
              "dcterms": "http://purl.org/dc/terms/",
              "myorg": "http://myorg.example.com/ontology#",
              "lastUpdated": "2024-03-15T10:00:00Z"
            }
            ```
        *   **JSON-LD Response Example:**
            ```jsonld
            {
              "@context": {
                "schema": "http://schema.org/",
                "dcterms": "http://purl.org/dc/terms/",
                "myorg": "http://myorg.example.com/ontology#",
                "lastUpdated": { "@id": "http://purl.org/dc/terms/modified", "@type": "xsd:dateTime" }
              },
              "lastUpdated": "2024-03-15T10:00:00Z"
            }
            ```
*   **Federation and Mirroring:** For robustness and availability, the creation of federated or mirrored registries is encouraged.
*   **Official vs. Private Registries:** There may be an "official" registry for common UCL prefixes (maintained by a UCL standards body), alongside private or domain-specific registries managed by organizations or communities.

### 6.2. Policy for Handling Extension Collisions and UCL-ID Resolution

When a UCL-ID (full URI) might be defined or referenced by multiple sources or vocabularies, or when a prefix could map to different base URIs, a clear resolution policy is needed:

1.  **Local `@prefix` Declarations in Message:** These have the **highest priority**. An `@prefix` declaration within a message overrides any other definition of that prefix for the scope of that message.
2.  **Predefined Prefixes from UCL Specification:** These have the next priority (e.g., `ucl:`, `rdf:`, `xsd:`).
3.  **Configured Prefix Registries:** If a prefix is not resolved locally or as predefined, configured external registries are queried in the order of preference specified in the receiving system's configuration.
4.  **Semantics of UCL-IDs (Full URIs):**
    *   A URI uniquely identifies a resource. If two different vocabularies use the same URI to define a concept, they are assumed to be referring to the same thing (Linked Data principle).
    *   If two vocabularies use different URIs for similar but non-identical concepts, they are considered distinct concepts. Mapping or aligning these concepts (e.g., via `owl:sameAs`, `skos:exactMatch`) is the task of ontologies and reasoning systems, not the basic UCL parser.
    *   If a receiving system encounters a UCL-ID (URI) for which it has no local or accessible semantic definition, it *may* attempt to dereference the URI to obtain a description (e.g., an RDF/OWL document, JSON-LD, or an HTML page with embedded metadata). This behavior is optional and depends on the system's capabilities.
5.  **Unresolvable Ambiguity:** If, after following the above steps, a UCL-ID remains ambiguous or a prefix cannot be resolved, the parser **must** signal an error (e.g., `ucl:errCode:UnresolvedIdentifier` or `ucl:errCode:AmbiguousIdentifier`).

### 6.3. Extending UCL: Defining New Vocabularies and Contexts

UCL is designed to be extensible through the creation and use of new UCL-IDs (URIs) organized into vocabularies or ontologies.

*   **Creating New Terms:**
    1.  **Define a Namespace (Namespace URI):** Choose a base URI for the new vocabulary (e.g., `http://mycompany.com/ontology/hr#`).
    2.  **Define Terms:** Create URIs for each new concept, property, action, context, etc., within that namespace (e.g., `http://mycompany.com/ontology/hr#Employee`, `http://mycompany.com/ontology/hr#hasSalary`).
    3.  **Document Semantics:** Crucially, document the meaning of each new term. This **should** be done using semantic web standards (RDF Schema, OWL) or at least machine-readable HTML documentation (e.g., with embedded RDFa or JSON-LD metadata) available at the term's or namespace's URI. Clear textual specification is also a starting point.
    4.  **Register Prefix (Optional but Recommended):** Register a short prefix for the new namespace (e.g., `myhr:`) in an accessible prefix registry to facilitate its use.
*   **Using New Terms:** Once defined and documented, new UCL-IDs can be used in UCL messages, typically with their registered or locally declared prefix.
*   **Domain-Specific Contexts:** Contexts (`ContextStackPart`) are the primary mechanism for indicating that a UCL message is using terms from a particular vocabulary or domain. For example, `# myhr:context:PersonnelAction / ...` would indicate that terms in the payload should be interpreted according to the `myhr:` ontology.

### 6.4. Interoperability

Interoperability in UCL is based on:

*   **Common Syntax:** All UCL-implementing systems adhere to the same grammar.
*   **Global Identifiers:** The use of URIs as the foundation for UCL-IDs.
*   **Shared Vocabularies:** The encouragement to reuse existing standard vocabularies (Schema.org, Dublin Core, FOAF, SKOS, domain-specific ones like FHIR for healthcare, etc.) where possible.
*   **Explicit Contextualization:** The context stack helps clarify which vocabulary or interpretation applies.
*   **Semantic Mapping:** For systems using different vocabularies for similar concepts, interoperability requires mapping between these vocabularies, which is a typical task of ontology engineering (e.g., using `owl:equivalentProperty`, `skos:mappingRelation`). UCL provides the language to *use* these terms, but mapping is a separate process.

## Part 7: Canonical Binary Serialization (High-Level Overview) - "UCL-Bin"

### 7.1. Purpose and Principles of Binary Serialization

While the textual form of UCL 4.2 is optimized for human readability, debugging, and specification, a Canonical Binary Serialization (conceptually referred to herein as "UCL-Bin") is defined to:

*   **Transmission Efficiency:** Significantly reduce the size of UCL messages for network transmission.
*   **Storage Efficiency:** Allow for more compact storage of UCL data.
*   **Parsing Speed:** Enable faster machine-to-machine parsing compared to complex text string analysis.
*   **Unambiguity:** Maintain a unique and lossless mapping to UCL's Canonical Textual Form.

**Principles of UCL-Bin:**

1.  **1:1 Mapping with Canonical Textual Form:** It must be possible to convert a UCL-Bin message to its exact Canonical Textual Form and vice-versa, without loss of semantic information.
2.  **Compactness:** Utilize variable-length encodings, string/UCL-ID tables (interning), and native binary representations for data types.
3.  **Parsing/Serialization Speed:** The format must be designed for efficient processing.
4.  **Extensibility:** The binary format should accommodate future extensions to the UCL language.

### 7.2. High-Level Structure of a UCL-Bin Message (Conceptual)

A UCL-Bin message could be structured as follows. Exact details (e.g., number of bytes for tags, field order) would require a formal binary specification document.

```text
+-------------------------------------------------------------------+
| Magic Number (e.g., 4 bytes, e.g., "UCLB")                        |
+-------------------------------------------------------------------+
| Header Flags (e.g., 1-2 bytes)                                    |
|   - UCL-Bin format version                                        |
|   - Compression flag (e.g., payload compressed with Gzip/Zstd)    |
|   - Endianness flag (if needed for certain numeric types)         |
|   - Other flags for future extensions                             |
+-------------------------------------------------------------------+
| [Optional] Locally Declared Prefix Table                          |
|   - Number of entries (VarInt)                                    |
|   - For each entry:                                               |
|     - Length of Prefix String (VarInt)                            |
|     - Prefix String (UTF-8 bytes)                                 |
|     - Length of Base URI String (VarInt)                          |
|     - Base URI String (UTF-8 bytes)                               |
+-------------------------------------------------------------------+
| [Optional] String/UCL-ID Table (String Interning)                 |
|   - Number of strings (VarInt)                                    |
|   - For each string:                                              |
|     - Length of string (VarInt)                                   |
|     - String (UTF-8 bytes)                                        |
|   (These strings/UCL-IDs can then be referenced by a compact      |
|    integer index in the message body)                             |
+-------------------------------------------------------------------+
| UCL Message Body (serialization of components)                    |
|   - [Type Tag + Data] SourcePart (if present)                     |
|   - [Type Tag + Data] Direction (if present and not default)      |
|   - [Type Tag + Data] TargetPart                                  |
|   - [Type Tag + Data] OperationVerb                               |
|   - [Type Tag + Data] UCLID_OperationSpecific                     |
|   - [Type Tag + Data] ModifiersPart (if present)                  |
|   - [Payload Presence Flag]                                       |
|   - [Type Tag + Data] PayloadPart (if present)                    |
|   - [Type Tag + Data] ContextStackPart                            |
+-------------------------------------------------------------------+
```

### 7.3. Encoding of Components and Data Types in UCL-Bin

*   **Type/Tag Bytes:** Each data element or structural component in UCL-Bin would be preceded by one or more type/tag bytes that identify its nature (e.g., UCL-ID, String, Integer, List, Map, etc.) and possibly length information.
*   **UCL-IDs:**
    *   Could be encoded as:
        1.  A reference to an index in the Prefix Table + a reference to an index in the String Table (for the `local_part`).
        2.  An index riferimento a "well-known" or predefined UCL-IDs.
        3.  A full URI string (less efficient, used if not internable).
*   **Strings:**
    *   Reference to an index in the String Table.
    *   Alternatively: Length (VarInt) followed by UTF-8 bytes.
*   **Numbers:**
    *   Integers: Variable-length encoding (VarInt) or fixed-size types (int8, int16, int32, int64).
    *   Decimals: IEEE 754 format (32-bit float, 64-bit double).
*   **Booleans and Null:** Dedicated single tag bytes.
*   **Lists:**
    *   "List" type tag.
    *   Number of elements (VarInt).
    *   Sequence of elements, each encoded according to its type.
*   **Maps:**
    *   "Map" type tag.
    *   Number of key-value pairs (VarInt).
    *   Sequence of pairs: Key (encoded as UCL-ID or String), Value (encoded according to its type).
*   **DateTime, Durations, Binary:**
    *   DateTime: Could be encoded as Unix timestamps (seconds or milliseconds from epoch, as int64) or ISO 8601 strings (if textual precision is crucial and string table not used).
    *   Durations: ISO 8601 strings (via string table) or a decomposition into numeric components.
    *   Binary: "Binary" type tag, Length (VarInt), followed by raw bytes.

### 7.4. String/UCL-ID Table (String Interning) Considerations

*   To reduce redundancy, strings and UCL-IDs (or their local parts/base URIs) that appear multiple times in a message can be stored once in a table at the beginning of the message (or in a session context) and then referenced by a compact integer index.
*   This is particularly useful for map keys, and common UCL-IDs for types or contexts.

### 7.5. Conceptual Example (Ultra-Simplified)

Textual UCL Message: `myprefix:ObjA > myprefix:ServiceB execute myprefix:Action1 : { myprefix:param1:"val1" } # myprefix:Ctx1`

Potential Binary Structure (very abstract):
```text
[MAGIC_UCLB]
[HEADER_FLAGS]
[PREFIX_TABLE_START]
  [NUM_PREFIXES=1]
  [LEN_PREFIX_STR="myprefix"] ["myprefix" bytes] [LEN_URI_BASE="http://..."] ["http://..." bytes]
[STRING_TABLE_START]
  [NUM_STRINGS=6] ; "ObjA", "ServiceB", "Action1", "param1", "val1", "Ctx1"
  [LEN="ObjA"] ["ObjA" bytes]
  ... (other strings)
[BODY_START]
  [TAG_UCLID] [PREFIX_IDX=0] [STRING_IDX_LOCAL_PART=0] ; Source: myprefix:ObjA
  [TAG_DIRECTION_DEFAULT_OR_EXPLICIT_>] 
  [TAG_UCLID] [PREFIX_IDX=0] [STRING_IDX_LOCAL_PART=1] ; Target: myprefix:ServiceB
  [TAG_OPERATIONVERB_EXECUTE]
  [TAG_UCLID] [PREFIX_IDX=0] [STRING_IDX_LOCAL_PART=2] ; OperationID: myprefix:Action1
  [TAG_PAYLOAD_PRESENT_FLAG]
  [TAG_MAP_START]
    [NUM_PAIRS=1]
    [TAG_UCLID_AS_KEY] [PREFIX_IDX=0] [STRING_IDX_LOCAL_PART=3] ; Key: myprefix:param1
    [TAG_STRING_AS_VALUE] [STRING_IDX=4] ; Value: "val1"
  [TAG_MAP_END]
  [TAG_CONTEXTSTACK_START]
    [NUM_CONTEXTS=1]
    [TAG_UCLID] [PREFIX_IDX=0] [STRING_IDX_LOCAL_PART=5] ; Context: myprefix:Ctx1
  [TAG_CONTEXTSTACK_END]
```

### 7.6. Reference to External Specification

The detailed definition of the UCL-Bin format, including exact tag byte values, VarInt encoding rules, endianness handling, etc., **will be contained in a separate, complementary specification document to the UCL 4.2 textual specification.**

This section serves only to outline the expected principles and high-level structure for such a binary format.

---

## Part 8: LLM Interaction (Tooling and Guidelines)

This section provides recommendations and guidelines to facilitate effective interaction between systems using UCL 4.2 and Large Language Models (LLMs), both for interpreting UCL and for generating UCL messages.

### 8.1. Role of LLMs in the UCL Ecosystem

LLMs can play several key roles within the UCL ecosystem:

1.  **"Prompt Compiler" from Natural Language to UCL:** Translating user requests formulated in natural language into the structured and unambiguous UCL 4.2 format.
2.  **Interpreter of UCL Messages:** Understanding a UCL message and performing the requested task (e.g., answering a query, generating content, executing a simulated action), especially if the LLM itself acts as the `TargetPart` of the message.
3.  **Generator of UCL Messages:** Creating UCL messages as output, for example, to communicate with other UCL-aware services or agents, or to structure its own internal "thoughts" or intermediate plans.
4.  **UCL Validator/Corrector (Limited):** An LLM could be used to identify common syntactic or semantic errors in a UCL message, especially if instructed with the rules, though a formal parser would remain more reliable for strict validation.
5.  **Assistant for Creating UCL Vocabularies/Ontologies:** Helping developers define new UCL-IDs and their semantic descriptions.

### 8.2. System Instructions (System Prompt) for LLMs

To enable an LLM to operate effectively with UCL (either as a "Prompt Compiler" or as an interpreter/generator), it is crucial to provide it with a clear and detailed **System Prompt** (or equivalent custom instructions) that defines:

*   Its specific role concerning UCL (e.g., "You are a translator from NL to UCL," "You are an agent that receives UCL commands and executes X").
*   A concise yet sufficient summary of the **key UCL 4.2 specifications** (message structure, `OperationVerb` types, payload syntax for lists and maps, use of `@prefix` and UCL-IDs, context handling).
*   **Concrete examples** of NL-to-UCL translations or UCL messages it is expected to receive/generate.
*   Guidelines on how to handle ambiguity, missing information, or errors.

*   **Example System Prompt Snippet for a "Prompt Compiler":**
    ```text
    System: YOU ARE A UCL PROMPT COMPILER. Your task is to translate user input in natural language into a valid UCL 4.2 message.
    Key UCL 4.2 Specifications to Follow:
    - Message: [Source] [Direction] Target OperationVerb OperationID_UCLID [: Payload] # ContextStack
    - OperationVerbs: query, execute, read, create, delete, subscribe, notify.
    - Payload: Maps { "keyUCLID": Value, ... }, Lists [ Value1, Value2, ... ]. Values can be Strings "...", Numbers, Booleans, ucl:val:null, UCL-IDs, or nested maps/lists.
    - UCL-IDs: Use `prefix:local_part`. Common prefixes: ucl:, schema:, wd:. Declare others with `@prefix p: <URI>`.
    - Context: `# context1 / context2`.
    When you receive user input, analyze it and generate ONLY the corresponding UCL 4.2 message. If the input is ambiguous, you may ask one clarifying question BEFORE generating the UCL.

    NL Example: "Find flights from Rome to Paris for tomorrow"
    UCL Output Example:
    @prefix schema: <http://schema.org/>
    @prefix trvl: <http://example.org/travel#>
    ucl:id:User > trvl:service:FlightSearch query trvl:verb:findFlights : { schema:departureAirport:wd:Q220, schema:arrivalAirport:wd:Q90, schema:departureDate:"YYYY-MM-DD" } # trvl:context:FlightBooking
    ---
    NOW TRANSLATE THE FOLLOWING USER INPUT:
    ```

### 8.3. Developing a "Prompt Compiler" (NL -> UCL)

This is a crucial tool for UCL adoption.

*   **LLM-Based Approach:** Using an LLM (as described in 8.2) is the most flexible approach. It requires a good system prompt and potentially fine-tuning on NL-UCL pairs.
*   **Hybrid Approach:** Combine more traditional NLP techniques (NER, intent classification, dependency parsing) to extract a preliminary structure, and then use an LLM to refine, resolve ambiguities, and map to specific UCL-IDs.
*   **Considerations:**
    *   **Ambiguity Handling:** The compiler must have strategies for when NL input is unclear (ask for clarification, provide UCL options).
    *   **Vocabulary Knowledge:** The compiler needs access to (or knowledge of) vocabularies and ontologies to map NL terms to correct UCL-IDs.
    *   **Interactivity:** A good compiler might be interactive, guiding the user to specify their request better.

### 8.4. UCL Generation by LLMs

When asking an LLM to *generate* a UCL message (e.g., as the output of an action, or as an internal action plan):

*   **Clear Instructions:** The prompt requesting UCL generation must be clear about the expected UCL format.
*   **Provide Examples (Few-Shot):** Including examples of valid UCL messages in the prompt can significantly improve the quality and syntactic correctness of the generated UCL.
*   **Post-Generation Validation:** UCL generated by an LLM should ideally be validated by a formal UCL parser before being utilized.

### 8.5. Tokenization Considerations

*   **Standard Tokenization:** Initially, reliance will be on the LLM's standard tokenizer. The UCL 4.2 syntax (with textual verbs, clear separators, and a structure reminiscent of JSON/code) is designed to be reasonably well-tokenized by current models.
*   **Custom UCL Tokenizer (Advanced Optimization):** To maximize efficiency (reduce token count per UCL message) and potentially improve the LLM's structural understanding, a UCL-specific tokenizer *could* be considered. This is a significant effort and would likely be a later step once UCL is more established.
    *   A custom tokenizer might treat `ucl:id:`, `schema:name`, `execute`, `#` as single tokens or optimized token sequences.

### 8.6. Relaxed Parsing vs. LLM Output Validation

*   When an LLM generates UCL, it may not perfectly adhere to the Canonical Form.
*   The system receiving UCL from an LLM should ideally use a parser in **Permissive Mode** that normalizes the input to Canonical Form and reports warnings for deviations.
*   For critical tasks, more rigorous validation of LLM-generated UCL is recommended.

### 8.7. Feedback Loop and Continuous Learning

*   If using an LLM as a "Prompt Compiler," implementing a feedback mechanism is beneficial. If the generated UCL is incorrect or doesn't capture the intent, this feedback can be used to refine the compiler's system prompt or to create data for future fine-tuning.

## Appendices

### Appendix A: Glossary of Terms

*   **ABNF (Augmented Backus-Naur Form):** A formal grammar notation used in this specification to define the syntax of UCL.
*   **AI Agent:** A software entity, often powered by an LLM or other AI techniques, that can perceive its environment, reason, and act to achieve goals.
*   **API (Application Programming Interface):** A set of rules and protocols for building and interacting with software applications.
*   **AST (Abstract Syntax Tree):** An internal tree representation of a parsed UCL message, reflecting its syntactic structure.
*   **Canonical Form (UCL):** The strict, normalized textual representation of a UCL message, serving as the reference for conformance.
*   **Context Stack (`ContextStackPart`):** An ordered list of UCL-IDs that provides the semantic and interpretive frame for a UCL message, read from most specific to most general.
*   **CURIE (Compact URI Expression):** A shortened form of a URI, using a prefix mapped to a base URI (e.g., `schema:Book`).
*   **IRI (Internationalized Resource Identifier):** A generalization of URI that allows a wider range of Unicode characters, defined in RFC 3987.
*   **JSON (JavaScript Object Notation):** A lightweight data-interchange format. UCL's map and list syntax in the payload is JSON-like.
*   **JSON-LD (JSON for Linked Data):** A method to embed Linked Data (RDF) semantics within JSON.
*   **LLM (Large Language Model):** An advanced AI model trained on vast amounts of text data, capable of understanding and generating human-like text and, pertinently, structured data like UCL.
*   **Literal Value (`LiteralValue`):** A fundamental data value in UCL, such as a string, number, boolean, null, date/time, duration, or binary data.
*   **Map (`MapValue`):** An unordered collection of key-value pairs in UCL, where keys are typically UCL-IDs or strings, and values can be any UCL `Value` type.
*   **Namespace (URI):** A base URI used in conjunction with a local part to form a globally unique UCL-ID. Often associated with a prefix.
*   **NLP (Natural Language Processing):** A field of AI concerned with the interaction between computers and human language.
*   **NLU (Natural Language Understanding):** A subfield of NLP focused on machine reading comprehension.
*   **Ontology:** A formal, explicit specification of a shared conceptualization. In the context of UCL, ontologies (like Schema.org, or custom ones) provide the definitions for UCL-IDs.
*   **Operation (`OperationPart`):** The component of a UCL message that specifies the primary action or intent, consisting of an `OperationVerb` and a specific `UCLID_OperationSpecific`.
*   **Parser:** A software component that analyzes a string of symbols (a UCL message) according to the rules of a formal grammar.
*   **Payload (`PayloadPart`):** The component of a UCL message containing the data or parameters associated with an operation, or the result of an operation.
*   **Permissive Mode (Parsing):** A UCL parsing mode that tolerates minor deviations from the Canonical Form and attempts to normalize them.
*   **Prefix (`@prefix` declaration):** An alias (e.g., `schema:`) declared in a UCL message to represent a base URI (e.g., `<http://schema.org/>`).
*   **Prompt Compiler (UCL):** A conceptual tool or LLM configuration designed to translate natural language requests into well-formed UCL messages.
*   **RDF (Resource Description Framework):** A World Wide Web Consortium (W3C) standard for representing information in the Web, often used as a foundation for semantic web technologies and a key inspiration for UCL's use of URIs.
*   **Registry (Prefix/UCL-ID):** A system (centralized or distributed) that stores mappings from prefixes to base URIs, or definitions for UCL-IDs.
*   **Semantic Web:** A W3C vision for a Web of linked data, where information is given well-defined meaning, better enabling computers and people to work in cooperation. UCL aligns with these principles.
*   **Serialization (Binary - UCL-Bin):** A compact, machine-optimized binary format for UCL messages, designed for efficient transmission and parsing.
*   **Strict Mode (Parsing):** A UCL parsing mode that requires strict adherence to the Canonical Form, rejecting any deviations.
*   **System Prompt (LLM):** Initial instructions given to an LLM to define its role, capabilities, and constraints for an interaction.
*   **UCL-ID (Universal Contextual Language Identifier):** A globally unique identifier, canonically a URI (or IRI), used in UCL to refer to any resource (entity, type, property, action, context, etc.).
*   **URI (Uniform Resource Identifier):** A compact sequence of characters that identifies an abstract or physical resource, as defined in RFC 3986.
*   **Value (UCL):** The general term for any data element in UCL, which can be a `UCLID`, `LiteralValue`, `ListValue`, or `MapValue`.
*   **Vocabulary:** A defined set of UCL-IDs (terms) and their meanings, typically associated with a specific namespace or ontology.

### Appendix B: Complete ABNF Grammar for UCL 4.2 (Textual Canonical Form)

This ABNF provides a consolidated view of the UCL 4.2 grammar. `CANONICAL_SPACE` implies `%x20`.

```abnf
UCLMessage      = [PrefixDeclarations] [SourcePart CANONICAL_SPACE] [Direction CANONICAL_SPACE] TargetPart CANONICAL_SPACE OperationPart [CANONICAL_SPACE ModifiersPart] [CANONICAL_SPACE PayloadSeparator CANONICAL_SPACE PayloadPart] CANONICAL_SPACE ContextSeparator CANONICAL_SPACE ContextStackPart

PrefixDeclarations = *(PrefixDeclaration NEWLINE)
PrefixDeclaration  = "@prefix" CANONICAL_SPACE DeclaredPrefix ":" CANONICAL_SPACE "<" URI_Base ">"
DeclaredPrefix     = 1*(ALPHA / DIGIT / "_" / "-") ; Cannot start or end with "_" or "-"
URI_Base           = 1*<any char except ">"> ; Represents a valid absolute URI string
NEWLINE            = %x0A ; LF

SourcePart      = UCLID
Direction       = ">" / "<" / "<>"
TargetPart      = UCLID
OperationPart   = OperationVerb CANONICAL_SPACE UCLID_OperationSpecific
OperationVerb   = "read" / "execute" / "query" / "create" / "delete" / "subscribe" / "notify" / CustomVerb_UCLID
CustomVerb_UCLID= UCLID
UCLID_OperationSpecific = UCLID

ModifiersPart   = MODIFIER_PREFIX UCLID *(CANONICAL_SPACE MODIFIER_PREFIX UCLID)
MODIFIER_PREFIX = "^"

PayloadSeparator= ":"
PayloadPart     = Value

ContextSeparator= "#"
ContextStackPart= UCLID *(CONTEXT_STACK_SEPARATOR UCLID)
CONTEXT_STACK_SEPARATOR = "/"

; UCL-ID (Simplified for ABNF, actual validation is more complex: URI or CURIE)
UCLID           = PrefixedName / FullURIWithBrackets ; Or a Full URI without brackets if it doesn't contain spaces or conflicting chars.
PrefixedName    = Prefix ":" LocalPart
Prefix          = 1*(ALPHA / DIGIT / "_" / "-") ; Cannot start or end with "_" or "-"
LocalPart       = 1*(%x21 / %x23-%x2F / %x3A-%x40 / %x41-%x5A / %x5C / %x5E-%x7E / %x25 2HEXDIG) ; Printable chars excluding <, >, ", space, plus percent-encoded. Needs refinement for full URI path/query/fragment rules.
FullURIWithBrackets = "<" 1*<any char except ">"> ">" ; Full URI enclosed in angle brackets.

; Value Types
Value           = ListValue / MapValue / SingleValue ; Order matters for parser, try compound first
SingleValue     = UCLID / LiteralValue

LiteralValue    = NullLiteral / BooleanLiteral / NumberLiteral / DateTimeLiteral / DurationLiteral / BinaryLiteral / StringLiteral ; StringLiteral last due to its generic nature

NullLiteral     = "ucl:val:null" ; Predefined UCL-ID
BooleanLiteral  = "true" / "false"
NumberLiteral   = [SIGN] 1*DIGIT ["." 1*DIGIT] [("e" / "E") [SIGN] 1*DIGIT]
SIGN            = "+" / "-"
StringLiteral   = DQUOTE *CHAR_IN_STRING DQUOTE
CHAR_IN_STRING  = UNESCAPED_CHAR / ESCAPED_CHAR
UNESCAPED_CHAR  = %x20-21 / %x23-5B / %x5D-10FFFF ; Any char except DQUOTE, BACKSLASH
ESCAPED_CHAR    = BACKSLASH (DQUOTE / BACKSLASH / "b" / "f" / "n" / "r" / "t" / ("u" 4HEXDIG))
DQUOTE          = %x22
BACKSLASH       = %x5C

DateTimeLiteral = StringLiteral ; Content must conform to ISO 8601 date-time with timezone
DurationLiteral = StringLiteral ; Content must conform to ISO 8601 duration
BinaryLiteral   = "ucl:datatype:binary(" StringLiteral ")" ; StringLiteral here is Base64

ListValue       = "[" [Value *(LIST_SEPARATOR Value)] "]"
MapValue        = "{" [MapPair *(MAP_SEPARATOR MapPair)] "}"
MapPair         = MAP_KEY PAIR_SEPARATOR Value
MAP_KEY         = UCLID / StringLiteral

; Separators in Canonical Form for Lists/Maps
; For canonical form, a single space is used AFTER commas, and NO spaces around colons in map pairs.
; Example: [ "a", "b" ] and { "key":"value", "anotherKey":"anotherValue" }
LIST_SEPARATOR  = "," CANONICAL_SPACE
MAP_SEPARATOR   = "," CANONICAL_SPACE
PAIR_SEPARATOR  = ":"

; General definitions
ALPHA           = %x41-5A / %x61-7A   ; A-Z / a-z
DIGIT           = %x30-39            ; 0-9
HEXDIG          = DIGIT / "A" / "B" / "C" / "D" / "E" / "F" / "a" / "b" / "c" / "d" / "e" / "f"

