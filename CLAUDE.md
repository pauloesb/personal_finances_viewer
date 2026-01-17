# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal finances viewer application. (Update this description as the project develops.)

## Build & Development Commands

<!-- Add commands as the project is set up, e.g.: -->
<!-- npm install - Install dependencies -->
<!-- npm run dev - Start development server -->
<!-- npm test - Run tests -->
<!-- npm run lint - Run linter -->

## Architecture

<!-- Document the high-level architecture once established -->

---

## Prompting Rules (RLM-Inspired)

Based on "Recursive Language Models" (Zhang, Kraska, Khattab - MIT CSAIL, 2025), these rules improve handling of complex and long-context tasks.

<prompting-rules>

### Core Principle: Context as Environment

<rule name="external-context">
Treat large inputs as an external environment to interact with programmatically, not as something to process in one pass. Use tools to peek, filter, and decompose rather than attempting to hold everything in working memory.
</rule>

### Decomposition Strategies

<rule name="recursive-decomposition">
For complex tasks, break the problem into sub-tasks that can be solved independently:
1. Identify natural boundaries in the input (sections, files, documents)
2. Process each chunk with a focused sub-query
3. Aggregate results to form the final answer
</rule>

<rule name="chunking-strategy">
Before processing large inputs:
1. Probe the structure first (print first N lines, check format)
2. Identify logical chunking boundaries (headers, newlines, document separators)
3. Choose chunk sizes that preserve semantic coherence (~100-500K chars per chunk for sub-processing)
</rule>

### Filtering and Search

<rule name="filter-before-process">
Use code execution and model priors to filter context before deep processing:
- Regex queries to find relevant sections
- Keyword searches based on domain knowledge
- Print samples to understand structure before committing to a strategy
</rule>

<rule name="selective-attention">
Not all parts of the input are equally important. Identify high-value sections first:
- Search for keywords from the original query
- Use model priors about likely locations of answers
- Skip obviously irrelevant content programmatically
</rule>

### Verification Patterns

<rule name="verify-with-subcalls">
For critical answers, verify using targeted sub-queries with minimal context:
- Re-check key facts with fresh, focused queries
- Use code execution to programmatically verify computed answers
- Avoid context rot by keeping verification contexts small
</rule>

<rule name="build-up-answers">
For long-output tasks, construct answers iteratively:
1. Store intermediate results in variables
2. Process chunks and accumulate findings
3. Stitch outputs together programmatically
4. Return the built-up variable rather than generating all at once
</rule>

### Efficiency Rules

<rule name="batch-subcalls">
Minimize the number of sub-LM calls by batching related information:
- Aim for ~200K characters per sub-call when possible
- Group related questions into single queries
- Avoid one-call-per-line patterns for large datasets
</rule>

<rule name="complexity-aware">
Task complexity affects how much context can be effectively processed:
- Simple lookups (needle-in-haystack): Can handle very long contexts
- Linear aggregation (counting, summarizing): Moderate context limits
- Quadratic tasks (pairwise comparisons): Requires aggressive chunking
Adjust strategy based on information density requirements.
</rule>

### Anti-Patterns to Avoid

<rule name="avoid-full-context-loading">
Never attempt to process extremely long inputs in a single pass. Signs you need decomposition:
- Input exceeds ~100K tokens
- Task requires examining most/all of the input
- Multiple pieces of information must be aggregated
</rule>

<rule name="avoid-redundant-verification">
While verification is good, avoid excessive re-verification loops:
- Don't reproduce correct answers multiple times
- Trust well-verified intermediate results
- Set clear stopping conditions for iterative processes
</rule>

</prompting-rules>

### Example Patterns

```
# Pattern: Probe → Chunk → Process → Aggregate
1. print(context[:1000])  # Understand structure
2. chunks = split_by_logical_boundary(context)
3. results = [process_chunk(c) for c in chunks]
4. final = aggregate(results)

# Pattern: Filter → Focus → Answer
1. relevant = regex_search(context, keywords)
2. focused_context = extract_relevant_sections(relevant)
3. answer = query_with_focused_context(focused_context)
```
