# Building a "UCL Prompt Compiler" with an LLM

While UCL (Universal Contextual Language) provides a powerful, structured way to communicate with AI, writing complex UCL messages manually can be verbose and require familiarity with its syntax and relevant UCL-IDs. A "UCL Prompt Compiler" – essentially an LLM configured to translate natural language requests into well-formed UCL 4.2 messages – can bridge this gap, making UCL's power accessible through natural language.

This document provides a conceptual guide on how you might set up an LLM to act as such a compiler.

## Concept Overview

The core idea is to provide an LLM with:
1.  **The UCL 4.2 Specification:** Or at least a comprehensive summary of its syntax, core components, and principles.
2.  **The User's Natural Language Request:** The input you want to translate.
3.  **A Task Instruction:** Telling the LLM to perform the translation.

The LLM then uses its understanding of natural language and its learned ability to follow instructions and patterns (from the UCL spec provided) to generate the corresponding UCL message.

## Steps to Configure an LLM as a UCL Compiler

### 1. Craft a Detailed System Prompt / Meta-Prompt

This is the most crucial step. The system prompt (or initial set of instructions given to the LLM) needs to effectively "teach" the LLM about UCL and its role as a compiler.

**Key elements to include in the System Prompt:**

*   **Role Definition:** Clearly state the LLM's role (e.g., "You are a UCL 4.2 Prompt Compiler. Your task is to translate natural language user requests into valid UCL 4.2 messages.").
*   **UCL Specification Summary:**
    *   **Overall Message Structure:** `[Source] [Direction] Target OperationVerb OperationID_UCLID [: Payload] # ContextStack`
    *   **Key Components:** Briefly explain Source, Direction (and its default), Target, OperationVerb (list standard ones), OperationID_UCLID.
    *   **Payload Syntax:** Explain Maps `{ "Key": Value, ... }` and Lists `[ Value1, ... ]`. Mention common literal types (Strings `""`, Numbers, Booleans `true`/`false`, `ucl:val:null`). Emphasize that Keys in Maps should ideally be UCL-IDs.
    *   **UCL-ID Format:** Explain `prefix:local_part` and the use of `@prefix` declarations for non-predefined prefixes. List key predefined prefixes (`ucl:`, `schema:`, `wd:`, `rdf:`, `xsd:`).
    *   **Context Stack Syntax:** `# Context1 / Context2 ...` and its purpose.
    *   **Output Format:** Instruct the LLM to *only* output the UCL message, or to clearly delineate it if additional explanatory text is allowed.
*   **Vocabulary Guidance (Important):**
    *   Instruct the LLM to use standard prefixes and UCL-IDs (like `schema:`, `wd:`) whenever appropriate for common concepts.
    *   For domain-specific concepts not covered by standard vocabularies, instruct it to create reasonable, descriptive UCL-IDs using a placeholder prefix (e.g., `custom:` or `app:`) or to indicate where a more specific UCL-ID would be needed.
*   **Handling Ambiguity:**
    *   Instruct the LLM on how to handle ambiguous natural language input. Options:
        *   Make a "best guess" and generate UCL, possibly noting the assumption.
        *   Ask one or two clarifying questions to the user *before* generating the UCL.
        *   Generate multiple UCL interpretations if ambiguity is high.
*   **Examples (Few-Shot Learning):** Provide 2-3 clear examples of natural language input and their corresponding correct UCL 4.2 output. This significantly helps the LLM understand the desired transformation.

    ```
    // Example within System Prompt:
    // NL Input: "Find the current weather in London."
    // UCL Output:
    // @prefix schema: <http://schema.org/>
    // @prefix geo: <http://www.w3.org/2003/01/geo/wgs84_pos#>
    // ucl:id:User > ucl:service:WeatherService query schema:WeatherForecast : { geo:location: "London, UK" } # ucl:context:WeatherInquiry
    ```

### 2. Provide the Natural Language Prompt

Once the LLM is configured with the system prompt, you provide the user's natural language request as the main input.

**Example User Input:**
`"I need a detailed project plan for a new mobile app for urban gardeners. It should have user profiles, a photo feed, messaging, and a map. Give me phases from ideation to launch, with deliverables and resources for each."`

### 3. Instruct the LLM to Generate UCL

The final part of your interaction with the "compiler" LLM is to ask it to perform the translation. If using an API, this is part of the request. In a chat interface, after the system prompt, your user input implicitly asks for the translation.

### 4. Review and Iterate

*   **Review the Generated UCL:** Check for syntactic correctness (though an LLM might make small errors) and, more importantly, semantic accuracy – does the UCL correctly capture the intent and details of the natural language prompt?
*   **Refine the System Prompt:** If the LLM consistently makes certain types of errors or misinterpretations, refine the system prompt. Add more examples, clarify UCL rules, or provide more specific guidance on vocabulary mapping.
*   **Consider a Feedback Loop:** For more advanced systems, you could implement a loop where the generated UCL is validated by a formal UCL parser, and if errors occur, feedback is given to the LLM (or a human) to correct it.

## Benefits of an LLM-Powered UCL Compiler

*   **Accessibility:** Makes UCL usable by those who don't want to learn its full syntax.
*   **Speed:** Potentially faster than manually crafting complex UCL messages.
*   **Leverages LLM Strengths:** Uses the LLM's NLU capabilities to interpret the user's intent.
*   **Adaptability:** The "compiler" can be updated by modifying its system prompt, without rewriting traditional code.

## Limitations

*   **Accuracy Depends on LLM and System Prompt:** The quality of the generated UCL is highly dependent on the LLM's capabilities and the clarity/comprehensiveness of the system prompt defining UCL.
*   **Potential for "Hallucinated" UCL-IDs:** The LLM might invent UCL-IDs or use them incorrectly if not sufficiently guided on vocabulary.
*   **Handling Extreme Complexity or Nuance:** Very complex or subtly nuanced natural language might still be challenging for an LLM to translate perfectly into UCL without ambiguity.

Despite limitations, using an LLM as a "UCL Prompt Compiler" is a promising approach to bridge the gap between the ease of natural language and the precision of a formal language like UCL, ultimately leading to more powerful and reliable AI interactions.

---

This concludes the initial set of documentation guides in the `docs/` folder.
Next, we would typically move to populating the `examples/` folder.