---
applyTo: '*'
---

# Rate Limit Prevention (CRITICAL)

**You cannot recover from rate limits - the chat session will stop completely. Prevention is essential. Do NOT ask for confirmation - work autonomously but cautiously.**

## Rules

- **Be concise**: Keep responses short and to the point. Avoid verbose explanations.
- **Don't repeat back**: Don't echo the user's request or repeat code they've already shared.
- **Plan first**: Before making ANY tool calls, create a complete mental plan. Identify the minimum number of calls needed.
- **Absolute minimum tool calls**: Only make tool calls that are strictly necessary.
- **Batch aggressively**: Combine multiple operations into single requests wherever possible.
- **Reuse information**: Never re-fetch data you already have from earlier in the conversation.
- **No exploratory calls**: No speculative searches or "just checking" requests.
- **Checkpoint silently**: After significant work, briefly note what's done so progress isn't lost if the session breaks.
- **If uncertain**: Make a reasonable decision and proceed - do not ask me to confirm.
- **Max 2 tool calls per response**: Never make more than 2 tool calls in a single response. Complete the response, then continue in the next one.

## Output Length Limits

**Responses that are too long will be cut off and fail. Stay within limits.**

- **Max ~300 lines per response**: If output would exceed this, split across multiple responses.
- **One file at a time**: When generating or editing multiple files, output one file per response, then continue.
- **Truncate examples**: Show only essential code snippets, not full files unless requested.
- **Use summaries**: For large changes, summarize what was done instead of showing everything.
- **Continue automatically**: If you need to split output, end with `... continuing` and proceed in the next response without asking.

## Visibility

When you take any rate-limit prevention action, output a short status line so I can monitor:

- `📦 BATCH: [what was combined]` - when combining multiple operations
- `♻️ REUSE: [what data]` - when reusing data instead of re-fetching
- `⏭️ SKIP: [what was skipped]` - when skipping an unnecessary call
- `💾 CHECKPOINT: [progress summary]` - when saving progress state
- `✂️ SPLIT: [what's being split]` - when breaking output into multiple responses
