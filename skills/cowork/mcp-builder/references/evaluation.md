# MCP Server Evaluation Guide

How to create evaluations that test whether agents can effectively use your MCP server.

## Purpose

Evaluations verify that your MCP server works well with LLM agents in practice — not just that individual tools return correct data, but that agents can compose tools to answer complex, realistic questions.

## Evaluation Process

### Step 1: Tool Inspection

List all available tools and understand their capabilities:
- What data can each tool access?
- What operations can each tool perform?
- How do tools compose together?

### Step 2: Content Exploration

Use **read-only** operations to explore available data:
- Run list/search tools to understand what data exists
- Note interesting patterns, relationships, and edge cases
- Identify data that requires multiple tool calls to answer questions about

### Step 3: Question Generation

Create **10 complex, realistic questions** that require:
- Multiple tool calls to answer
- Deep exploration of available data
- Reasoning across results from different tools
- Understanding of the domain

### Step 4: Answer Verification

For each question:
1. Solve it yourself using the MCP tools
2. Document the exact answer (single, verifiable value)
3. Verify the answer is stable (won't change over time)
4. Confirm the answer is unambiguous (one correct answer)

## Question Requirements

Each evaluation question must be:

| Requirement | Description |
|-------------|-------------|
| **Independent** | Not dependent on other questions |
| **Read-only** | Only requires non-destructive operations |
| **Complex** | Requires 3+ tool calls and reasoning |
| **Realistic** | Based on real use cases humans would care about |
| **Verifiable** | Single, clear answer verifiable by string comparison |
| **Stable** | Answer won't change over time (use historical data) |

### Good Question Examples

- "Which contributor has the most merged PRs in the last 6 months that touched files in the /src/auth directory?"
- "Find the issue with the most cross-references. What is its number?"
- "What is the average time-to-close for bugs labeled 'critical' in 2024?"

### Bad Question Examples

- "List all open issues" (too simple, just one tool call)
- "What's the latest commit?" (answer changes over time)
- "Is the code well-written?" (subjective, not verifiable)

## Output Format

Create an XML file with question-answer pairs:

```xml
<evaluation>
  <qa_pair>
    <question>Find the contributor who has opened the most issues labeled 'bug' in the past year. What is their username?</question>
    <answer>johndoe</answer>
  </qa_pair>
  <qa_pair>
    <question>Which repository has the highest ratio of closed issues to open issues? Report the owner/repo name.</question>
    <answer>vercel/next.js</answer>
  </qa_pair>
  <!-- 8 more qa_pairs -->
</evaluation>
```

## Running Evaluations

### Manual Testing

1. Start your MCP server
2. Connect via MCP Inspector or Claude Desktop
3. Ask each evaluation question
4. Compare agent's answer to expected answer
5. Score: exact match = pass, otherwise = fail

### Automated Testing

If using Claude Code SDK or similar:

```bash
# Run evaluation script
python run_eval.py --server ./my-mcp-server --eval eval.xml --model claude-sonnet-4-5-20250929

# Output: pass/fail per question + aggregate score
```

### Scoring

- **Pass rate**: Percentage of questions answered correctly
- **Tool usage efficiency**: Average tool calls per question (lower is better for same accuracy)
- **Error rate**: How often the agent encounters tool errors

## Evaluation Design Tips

1. **Start from data**: Explore your data first, then write questions based on what's interesting
2. **Test composition**: Best questions require combining results from multiple tools
3. **Include edge cases**: Empty results, pagination boundaries, special characters
4. **Vary difficulty**: Mix straightforward (3 tool calls) with hard (7+ tool calls)
5. **Domain-specific**: Questions should reflect how real users would query this service
6. **Stable answers**: Use historical data or fixed datasets to ensure answer stability
