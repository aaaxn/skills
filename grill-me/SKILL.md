---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when user wants to stress-test a plan, get grilled on their design, or mentions "grill me".
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one.

If a question can be answered by exploring the codebase, explore the codebase instead of asking me.

## How to ask questions

You MUST use the `AskUserQuestion` tool for every question you ask. Never output questions as plain text — always use the tool so I get an interactive prompt.

### Rules for using AskUserQuestion:

- Ask 1-4 questions per round using a single `AskUserQuestion` call.
- For each question, provide 2-4 option choices. One option should be your recommended answer — mark it with "(Recommended)" and put it first.
- Use the `description` field on each option to explain trade-offs or reasoning.
- Use the `header` field as a short category label (e.g., "Storage", "Auth", "Scope").
- Use `preview` when showing concrete alternatives like code snippets, schemas, or layouts.
- The user can always select "Other" to give a free-form answer, so don't add a generic "Other" option yourself.

### Interview flow:

1. Start by reading/exploring the relevant context (plan, codebase, conversation history).
2. Identify the first set of unresolved decisions.
3. Ask up to 4 questions per round via `AskUserQuestion`.
4. After receiving answers, briefly acknowledge the decisions, then move to the next set of unresolved questions.
5. Continue until all branches of the design tree are resolved.
6. When done, summarize all decisions made in a final overview.
