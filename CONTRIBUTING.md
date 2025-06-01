# Contributing to UCL (Universal Contextual Language)

Thank you for your interest in contributing to the UCL specification and ecosystem! We welcome contributions from everyone. 

There are many ways to contribute, from improving documentation and providing examples to proposing changes to the specification itself or developing tooling.

## How to Contribute

### 1. Reporting Issues / Bugs in the Specification

*   If you find a bug, an inconsistency, ambiguity, or an area for improvement in the UCL specification documents, please **open an issue** on the GitHub repository.
*   Clearly describe the issue, including:
    *   The specific part of the specification affected (e.g., section number, file name).
    *   Why you believe it's an issue or could be improved.
    *   Suggestions for resolution, if you have any.

### 2. Suggesting Enhancements or New Features

*   If you have an idea for a new feature, a change to an existing feature, or a way to make UCL better, please **open an issue** first to discuss it.
*   This allows for community discussion before significant work is undertaken.
*   Clearly outline the proposed enhancement, its motivation, and potential benefits or drawbacks.

### 3. Improving Documentation

*   Clear and comprehensive documentation is vital. If you find areas that are unclear, could be explained better, or are missing, please feel free to:
    *   Open an issue describing the documentation gap.
    *   Submit a pull request with your proposed documentation changes.
*   This includes the main `SPECIFICATION.MD`, files in the `docs/` folder, and comments within examples.

### 4. Providing More Examples

*   More examples of UCL messages for diverse use cases are always helpful.
*   You can submit new examples via a pull request to the `examples/` directory.
*   Please ensure your examples are well-commented, explaining their purpose and structure.

### 5. Developing Tooling (Parsers, Linters, Compilers)

*   The development of open-source tools (e.g., parsers for different programming languages, linters to check UCL syntax, "NL-to-UCL" compilers) is highly encouraged.
*   If you are working on such a tool, consider:
    *   Linking to it from this main UCL repository (e.g., in a "Related Tools" section in the README).
    *   Adhering to the UCL specification as closely as possible.

### 6. Participating in Discussions

*   Engage in discussions on open issues and pull requests.
*   Share your use cases, experiences, and insights.

## Pull Request Process

1.  **Fork the repository** and create your branch from `main`.
2.  If you've added code, add tests.
3.  If you've changed APIs or specifications, update the documentation.
4.  Ensure your contributions adhere to the project's coding style (if applicable) and aporlicensing.
5.  Make sure your commit messages are clear and descriptive.
6.  Open a **pull request** to the `main` branch of the original repository.
7.  Clearly describe your changes in the pull request description.
8.  Be prepared to discuss your changes and make adjustments based on feedback.

## Development Setup (for tooling, if applicable)

(This section would be filled in if the repository itself hosted code for a reference parser or other tools. For a spec-only repository, it might be minimal or absent initially.)

*   Example: "To contribute to the [hypothetical] Python reference parser, you'll need Python 3.8+ and Poetry for dependency management. Clone the repo and run `poetry install`..."

