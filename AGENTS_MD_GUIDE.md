# Optimizing `AGENTS.md` for AI Coding Agents

Based on [Vercel's research findings](https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals), this guide outlines the most effective way to structure `agents.md` files to maximize AI agent performance, specifically for framework-specific tasks.

## Key Findings

*   **Passive Context Wins:** Embedding a documentation index directly in `agents.md` achieved a **100% pass rate** on complex evaluations, compared to 53-79% for "skills" (tool-based retrieval).
*   **Zero Friction:** Agents often fail to invoke "skills" tools reliable. By placing the index in the system prompt (`agents.md`), the information is always available without the agent needing to make a decision to look it up.
*   **Retrieval-Led Reasoning:** The key is to force the agent to rely on local, version-matched documentation rather than its (potentially outdated) pre-training data.

## The Optimal Structure

To replicate the 100% success rate, your `agents.md` should follow this specific pattern:

### 1. The Directive
You must explicitly tell the agent to prioritize the provided documentation over its internal knowledge base.

> `IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning for any [Framework] tasks.`

### 2. The Local Docs Index
Instead of pasting full documentation (which bloats context), create a **compressed index** that points to local markdown files.

*   **Store Docs Locally:** Download framework documentation to a folder like `./.next-docs` or `./docs/reference`.
*   **Map the Files:** In `agents.md`, list the directory structure and file names in a compact format.

### 3. Compression Format (Pipe-Delimited)
Vercel recommends a pipe-delimited format to save token space (~8KB for a full framework index).

**Syntax:**
```text
[Header]|root: ./path/to/docs
|Directory:{file1.md,file2.md,...}
|Directory/SubDir:{file3.md,file4.md}
```

## `agents.md` Template

Here is the high-performance template you should use:

```markdown
# Agent Context & Rules

## Project Overview
[Brief description of your project]

## Documentation Index
[Next.js Docs Index]|root: ./.next-docs
|IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning for any Next.js tasks.
|01-app/01-getting-started:{01-installation.mdx,02-project-structure.mdx}
|01-app/02-building-your-application/01-routing:{01-defining-routes.mdx,02-pages-and-layouts.mdx}
|01-app/02-building-your-application/02-data-fetching:{01-fetching-caching-and-revalidating.mdx}
|... (continue for all relevant sections)
```

## Implementation Strategy

1.  **Scrape/Download Docs:** Use a tool or script to download the markdown (`.md` or `.mdx`) files of the documentation for the specific version of the framework you are using.
2.  **Place in Hidden Folder:** Store these files in a folder like `.docs/` or `.framework-docs/` in your project root so they don't clutter the workspace but are accessible to the agent.
3.  **Generate Index:** Create a script to scan that folder and generate the pipe-delimited index string.
4.  **Update `agents.md`:** Paste the generated index into your `agents.md` file.

## Why this works
1.  **No Ordering Issues:** The agent doesn't have to guess whether to "explore first" or "read docs first".
2.  **No Decision Fatigue:** The agent doesn't need to "decide" to use a tool; the map is right there.
3.  **Version Accuracy:** It prevents the agent from hallucinating APIs from older/newer versions by anchoring it to the specific files present in the repo.
