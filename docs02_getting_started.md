# Getting Started with UCL

This guide provides a quick start to understanding the basic structure of a UCL (Universal Contextual Language) message and how to begin thinking in terms of its core components. 

## Anatomy of a Simple UCL Message

Let's break down a basic UCL message. Imagine you want to ask an AI service to find information about a specific book. In UCL, this request might look like this:

```ucl
// 1. Prefix Declarations (Optional but Recommended)
@prefix schema: <http://schema.org/>
@prefix wd: <http://www.wikidata.org/entity/>
@prefix ucl: <http://ucl-spec.org/4.2/core#>

// 2. The UCL Message Itself
ucl:id:MyQueryAgent > ucl:service:KnowledgeBase query schema:Book : 
  { 
    schema:name: "1984", 
    schema:author: wd:Q3395 
  } 
# ucl:context:LiteraryWork / ucl:context:InformationRequest


Let's dissect this:

@prefix schema: <http://schema.org/>

What it is: A Prefix Declaration.

Purpose: It creates a shortcut. Instead of writing the full web address (URI) http://schema.org/Book every time, we can now write schema:Book. schema: is the prefix, and <http://schema.org/> is its corresponding base URI.

UCL has some predefined prefixes like ucl:, and often schema: and wd: are understood by systems even if not declared in every message, but explicit declaration is good practice for custom prefixes.

ucl:id:MyQueryAgent

Component: SourcePart (Optional).

Meaning: This is who is sending the message (the "Source"). It's a UCL-ID, a unique identifier. Here, ucl:id: might be a prefix for generic instance identifiers.

>

Component: Direction (Optional).

Meaning: Indicates the message flows from the Source to the Target. It's like an arrow.

ucl:service:KnowledgeBase

Component: TargetPart (Mandatory).

Meaning: This is who the message is for (the "Target" or recipient). Here, it's an AI service that acts as a knowledge base.

query schema:Book

Component: OperationPart (Mandatory).

Meaning: This specifies what to do.

query: The OperationVerb, indicating a request for information.

schema:Book: The UCLID_OperationSpecific, specifying that the query is about entities of type "Book" (as defined by Schema.org).

:

Component: PayloadSeparator (Mandatory if Payload is present).

Meaning: Separates the operation from its data/parameters.

{ schema:name: "1984", schema:author: wd:Q3395 }

Component: PayloadPart (Optional, but present here).

Meaning: This is the data associated with the operation. It's a "Map" structure (like a dictionary or JSON object).

schema:name: "1984": A key-value pair. The key schema:name (the "name" property from Schema.org) has the string value "1984".

schema:author: wd:Q3395: Another key-value pair. The key schema:author has the value wd:Q3395. wd:Q3395 is a UCL-ID representing George Orwell on Wikidata.

The payload provides the specific criteria for the book query.

#

Component: ContextSeparator (Mandatory).

Meaning: Separates the main message body from the context information.

ucl:context:LiteraryWork / ucl:context:InformationRequest

Component: ContextStackPart (Mandatory, at least one context).

Meaning: This tells the AI the "frame of mind" or domain in which to interpret the request.

ucl:context:LiteraryWork: The request pertains to a literary work.

/: Separates contexts in the stack.

ucl:context:InformationRequest: The overall nature of the interaction is an information request.

Contexts are read from most specific (left) to most general (right).

Thinking in UCL: From Natural Language to UCL

How would you get from "Find me info on the book 1984 by George Orwell" to the UCL above?

Identify the Core Action: "Find info" -> This is a query.

Identify the Main Subject/Type: "book" -> This can be mapped to schema:Book.

Identify the Target: Who performs this action? -> A ucl:service:KnowledgeBase.

Extract Key Details/Parameters:

"1984" -> This is the schema:name.

"George Orwell" -> This is the schema:author. Try to find a unique ID for him, like wd:Q3395.

Determine the Context: The request is about a ucl:context:LiteraryWork and it's an ucl:context:InformationRequest.

Assemble: Put it all together using UCL syntax.

Key Takeaways for Getting Started

Structure is Key: UCL organizes requests into clear components.

Unique Identifiers (UCL-IDs/URIs) are Powerful: They remove ambiguity (e.g., wd:Q3395 is definitively George Orwell). Prefixes make them shorter.

Context Matters: The #ContextStack guides the AI's interpretation.

Payload Carries the Data: Lists [...] and Maps {...} structure the specific parameters or data.

This is a very basic introduction. As you explore further, you'll see how UCL can represent much more complex requests and information.

Next, learn about:

Core Concepts of UCL

UCL Syntax in More Detail

---

