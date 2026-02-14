# Project Goal

This project builds a local reference of Effect documentation that is optimized for LLM usage.

The documentation content is extracted from `llms-full.txt`, then organized into clean, searchable notes that are easier for language models to retrieve and reason over.

# Core Intent

- Keep Effect documentation local and easy to navigate.
- Preserve technical accuracy while improving structure for LLM retrieval.
- Organize content in a consistent hierarchy that supports both humans and AI workflows.

# Documentation Structure

- Group documentation into nested sections: `major section -> subsection`.
- Follow the same grouping pattern already used in the current documentation layout.
- Keep related topics together so navigation and LLM retrieval remain predictable.
- Whenever a documentation file is generated, update `completed-subsections.md` in the project root.
- In that tracker, record completion under the correct `major section -> subsection`.

# Major Sections

Treat the following as major sections (top-level groups) for documentation organization:

- `getting-started`
- `error-management`
- `requirements-management`
- `resource-management`
- `observability`
- `runtime`
- `scheduling`
- `state-management`
- `batching`
- `caching`
- `concurrency`
- `stream`
- `sink`
- `testing`
- `code-style`
- `data-types`
- `traits`
- `schema`
- `ai`
- `micro`
- `platform`
