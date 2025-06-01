# UCL-IDs, Prefixes, and Namespaces

At the heart of UCL's ability to convey precise, unambiguous meaning are **UCL-IDs (Universal Contextual Language Identifiers)**. This document explains what they are, how they are constructed using URIs and prefixes, and why they are crucial for semantic interoperability.

Refer to Part 2 of the [UCL Specification](../SPECIFICATION.MD) for the formal rules.

## What is a UCL-ID?

A UCL-ID is a unique identifier for any "resource" or concept within the UCL ecosystem. This includes:

*   **Entities:** Specific things, like `wd:Q42` (Douglas Adams on Wikidata) or `myproduct:ID12345`.
*   **Types/Classes:** Categories of things, like `schema:Book` or `ucl:type:FinancialTransaction`.
*   **Properties/Parameters:** Attributes or characteristics, like `schema:author` or `ucl:param:quantity`.
*   **Actions/Verbs:** Operations to be performed, like `ucl:action:createOrder` or `ucl:verb:findEntities`.
*   **Contexts:** Interpretive frames, like `ucl:context:MedicalRecord` or `app:context:DevelopmentMode`.
*   **Predefined Values:** Enumerated values, like `ucl:val:null` or `ucl:value:Ascending`.

The key idea is that each UCL-ID should have a **globally unique and stable meaning**.

## Canonical Form: URIs (Uniform Resource Identifiers)

The standard, canonical form of a UCL-ID is a **URI (Uniform Resource Identifier)**, as defined in RFC 3986. IRIs (Internationalized Resource Identifiers, RFC 3987) are also encouraged for non-ASCII characters, which are then mapped to URIs for transport.

*   **Why URIs?**
    *   **Global Uniqueness:** URIs are designed to be globally unique. `http://schema.org/Book` refers to the same concept энергияno matter who uses it.
    *   **Resolvability (Dereferencing):** URIs can often be "looked up" on the web to find a definition or more information about the resource they identify (e.g., an HTML page, an RDF document, a JSON-LD description). This is a core principle of Linked Data.
    *   **Foundation for Semantics:** They provide a stable anchor for defining the meaning of terms in shared vocabularies and ontologies.

**Examples of UCL-IDs as full URIs:**
*   `http://schema.org/Person`
*   `http://www.wikidata.org/entity/Q5398422` (Mount Grappa)
*   `http://purl.org/dc/terms/title`

## Compact Form: CURIEs (Prefixed Names)

Full URIs can be long and make UCL messages verbose. To improve readability and conciseness, UCL supports a compact form similar to **CURIEs (Compact URI Expressions)**.

*   **Syntax:** `prefix:local_part`
    *   `prefix`: A short alias (e.g., `schema`, `wd`, `myont`).
    *   `local_part`: The remainder of the identifier specific to that namespace (e.g., `Book`, `Q5398422`, `MyCustomTerm`).

*   **Example:**
    *   `schema:Person` (instead of `http://schema.org/Person`)
    *   `wd:Q5398422` (instead of `http://www.wikidata.org/entity/Q5398422`)

## `@prefix` Declarations: Defining Your Shortcuts

For a parser to understand a compact `prefix:local_part` UCL-ID, the `prefix` must be mapped to its full base URI. This is done using `@prefix` declarations at the beginning of a UCL message.

*   **Syntax:** `@prefix declared_prefix: <FULL_URI_BASE>`
    *   One declaration per line, each followed by a newline.
*   **Example:**
    ```ucl
    @prefix schema: <http://schema.org/>
    @prefix geo: <http://www.w3.org/2003/01/geo/wgs84_pos#>
    @prefix myapp: <http://example.com/my-application-ontology#>

    // Later in the message:
    // myapp:User > geo:service:Locator query schema:Place ...
    ```
*   **Scope:** `@prefix` declarations are local to the message in which they appear.

### Predefined Prefixes in UCL 4.2

UCL 4.2 predefines a small set of common prefixes that **do not** require explicit `@prefix` declaration in every message. These include:

*   `ucl:` (for core UCL terms like `ucl:val:null`, `ucl:context:Generic`)
*   `rdf:`
*   `rdfs:`
*   `xsd:`
*   `owl:`
*   `schema:` (common convention)
*   `wd:` (common convention)

For all other namespaces, explicit `@prefix` declaration is required for clarity and to avoid reliance on parser-specific "well-known" prefix configurations beyond the core set.

## Resolution Order for Prefixes

When a UCL parser encounters a `prefix:local_part`:
1.  It first checks the **local `@prefix` declarations** within the current message.
2.  If not found, it checks the **UCL predefined prefixes**.
3.  If still not found, it *may* (if configured) consult **external prefix registries** (with caching).
4.  If the prefix cannot be resolved, it's an error.

## Namespaces and Vocabularies/Ontologies

*   A **namespace** is essentially the "domain" or "scope" defined by a base URI. All terms starting with `http://schema.org/` belong to the Schema.org namespace.
*   A **vocabulary** or **ontology** is a collection of terms (UCL-IDs/URIs) with their defined meanings and relationships, typically associated with one or more namespaces.
    *   **Examples:** Schema.org provides a vocabulary for common web entities. FOAF (Friend of a Friend) is an ontology for describing people and their relationships.
*   **UCL encourages the reuse of existing, well-known vocabularies** like Schema.org, Dublin Core (dcterms), FOAF, SKOS, etc., to promote interoperability.
*   You can (and often will need to) **define your own custom vocabularies/ontologies** for domain-specific concepts, actions, and contexts, identified by your own namespace URIs and prefixes.

## Versioning UCL-IDs

To manage changes in the meaning or structure associated with a UCL-ID over time, versioning is important.
*   **Recommended Convention:** `UCLID_Base@version_string` (e.g., `myapp:Order@2.1`).
*   Semantic Versioning (SemVer: MAJOR.MINOR.PATCH) is recommended for the `version_string`.
*   This allows systems to specify or request particular versions of a concept.

By leveraging URIs and a clear prefix mechanism, UCL-IDs provide a robust foundation for creating messages that are not only structured but also carry precise, globally understandable (when vocabularies are shared or mapped) semantic meaning.

---

Next, we'll look at:
*   [Overview of UCL's Binary Serialization (UCL-Bin)](./08_binary_serialization_overview.md)