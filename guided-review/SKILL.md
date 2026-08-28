---
name: guided-review
description: Guided review especially when an agent does so much work you don't have a deep understanding of all the changes, every single line of syntax etc. Force a complete understanding.
---

# Guided Review

A lot of work was done and the human operator wants to ensure they understand and vouch for every single:

- technical choice
- architectural choice
- line of code
- sytax sugar (style)
- layout/structure
- degree of abstraction
- patterns used

But personally, I am not that smart to just read a file tree alphabetically ordered diff and get up to speed instantly.

You are going to do a guided review such that we can understand small, ordered chunks to get on the same page. Its a common joke in tech - 5 line PR has 10 comments, 5000 line PR is LGTM, and merged.
Our goal is to avoid that LGTM issue, and rather break our entire big (or small) PR into semantically ordered chunks.

<example_description>

For example a large PR refactoring a payment system might include the followed ordered chunks:

1. Describe that based on an email from the CEO we must add an additional 10% charge on every line item. Our current architecture is fighting against this need - so now we need to restructure a bit

(see how we first got the business reason and the actual user facing context - now the reviewer/human understands the why)

2. Okay so given we use [blah] structure, we could consider options (A), (B), (C). This PR uses (B) because ...

(see how we got baseline (PR target branch current state) and options, then the chosen one - reviewer might suggest otherwise)
(it could help to show stack traces/call sites/touch points in a tiny ascii art style - tiny visualisation tricks help. A wall of text will overload the user! An image (or sketch or tree or structure etc ) says 1000 words as they say.)

3. So - following TDD we added this test that failed - with the correct message: > ''

(I'm going to tldr; the rest of the guided chunks now)

?. We checked that existing tests {did/didn't} cover the areas we need to refactor, so ...
?. Now the basic new pattern was added
?. Okay and finally the surcharge was added here and here, which caused a clean diff

</example_description>

Note - you can actually suggest improvements here - we all make mistakes. Time is not an issue - correctness, readability and long term maintainability is key here.
For example Linus famously notes that slightly more complex looking code is preffered, (given 2 options that do the same input + output), provided there is one less branch. (see how the reduced branching also reduces cognitive load)
We also care about ways that move towards pit of success - for example strongly typed IDs, the typecheck/compiler should yell at you if you accidentally pass ProductID when the parameter expects a CardItemID.

<guided_review_structure>

- use 100x less text than you would usually. Simple is king. If we cant explain it simply, perhaps our PR is overly complex when a more simple solution is *actually* the smarter option. (Code that looks stupidly easy is actually incredibly difficult)
- start with the why/what of the PR. A user might have just pasted in a GitHub PR link and have no idea what something is. Or maybe a colleagues WIP branch before they went on holiday. Context is key.
- code in general is an unfortunate means to an end. If the reviewer doesn't know the end the guided review failed its job.
- now semantic meaning/chronological order that builds shared understanding is key.
- its key to also show the git diff/churn. start with the total (in that first context message) then each chunk will say how many lines that is. e.g. (+50 -15 | 5% of the [PR/branch/?])
- the user must individually approve each chunk. that way its not a stupid hand wave github PR.
- you can call out key decisions or important/load bearing lines of code in each chunk - that is great since it helps the reviewer cover everything! 
- never use the question tool, the user will respond manually to all things

</guided_review_structure>

The user may have added extra info they think would be userful:

<user_context>

$ARGUMENTS

</user_context>
