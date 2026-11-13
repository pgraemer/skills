---
name: friction-driven-development
description: Improve a product by letting someone without inside knowledge try a real task through a real surface (a web UI, a CLI, an API, an MCP server, an SDK, the documentation, a setup guide), and letting the friction they hit decide what gets built next. Each friction point is verified, fixed with the smallest change, and confirmed by another attempt. Use when the user asks for friction-driven development, wants to find out why something is hard to use, wants a usability or onboarding test with fixes, or says things like "try this as a new user and fix what gets in the way".
---

# Friction-driven development

In friction-driven development the backlog is not a plan. It is whatever stopped a real attempt to get something done. One loop:

1. Someone who does not know the internals tries one task through one surface.
2. Every point where they stall, guess, or conclude something wrong is recorded.
3. Each point is verified against the product and fixed with the smallest change that removes it.
4. The task is attempted again. The loop ends when it goes through without friction.

Nothing is fixed that nobody stumbled over, and nothing counts as fixed until an attempt gets past it.

## What you need before starting

Settle these with the person who asked. Ask only for what you cannot find out yourself.

- **The task.** One concrete outcome in the user's words, for example "invite a colleague and give them read access". Not a feature list. Keep it verbatim.
- **The surface.** Exactly one way in: the web app, the CLI, the public API, the MCP server, the README, and so on. Comparing two surfaces means two runs with the same task.
- **The user.** Who is trying: what they know, what they do not know, what they have at hand. Default: competent in their field, new to this product, with only what the product itself hands them.
- **How to run it.** How the product is started and reached, and how sign-in works.
- **Where the truth is.** What you may read to check a claim: code, logs, database, job state.
- **What may change.** Which parts of the code or documentation may be changed freely during the run, and which need a decision first.
- **What is expensive.** Actions that cost money, take long, touch shared or production resources, or cannot be undone.
- **The fix budget.** How many fixes this run may make. Default: 3.

If the project keeps these answers in a file, read it first, and offer to write them down after the run so the next one starts faster.

## Roles

Keep these four apart. The method stops working when one role leaks into another.

- **Owner:** the person who asked. Decides everything expensive, irreversible, or outside what may change freely.
- **User:** tries the task. Has the task, the persona and the surface, and nothing else.
- **Observer:** you. Sets up the run, watches, checks claims, decides what gets fixed, writes the report. Never does the task for the user and never hints.
- **Builder:** makes the fixes. Knows the code, and never talks to the user.

The user must be a separate mind: a separate agent with its own context if the environment can start one, otherwise the owner or another person, who tries the task while you observe. You cannot play the user yourself. You have seen the code, so you would find things a real user cannot. If no separate user is available, say so and stop.

Observer and builder may be the same session when no second agent is available. The user must still learn nothing about what was changed.

## Briefing the user

Give the user the persona, the task verbatim, the surface, and these rules. Give no identifiers, names, paths or hints beyond the task.

- Use only the surface. No reading the source code, no database, no other way in. In a UI, find your way like a person does: menus, titles, buttons. No typed-in addresses you have not seen.
- Before anything that creates, changes, deletes, starts or sends something, say what you intend and why, and wait for approval.
- After each step, say what you did and what you saw.
- Report every point of friction: anything missing, hard to find, confusing, wrong, slow, too much, badly described, or without feedback.
- If you are stuck, say what you tried, where you looked and what you needed. Do not guess identifiers and do not work around a gap.
- When told to wait, wait. When told to try again, try again the way you normally would.
- At the end, say whether the task worked, and list the friction points, the worst first.

## The loop

### Approving actions

Approve what is cheap, local, reversible and part of the task. Approve harmless mistakes as well: a dead end is a finding. Take anything on the expensive list, any delete, and anything outside the task to the owner, with your own assessment, and pass their decision on unchanged.

### Do not help

A real user has nobody who corrects them. When the user concludes something wrong, let it stand and record where it leads. Step in only when a wrong conclusion is about to cause an expensive or irreversible action, or when the user goes in circles without learning anything new. Note every time you stepped in, and why.

### Verify, then classify

What the user reports is a symptom. Before it becomes a fix or a finding, check it against the product, and name its kind:

| Kind | Meaning | Usual fix |
|---|---|---|
| Missing | The capability does not exist. | Build it, or say clearly that it does not exist. |
| Hidden | It exists, but the user did not find it. | Naming, placement, a pointer where the user looked. |
| Broken | It exists and fails. | Repair it. |
| Misleading | The product's output led to a wrong conclusion, such as a cut-off list read as complete. | Change what the product says. |
| Heavy | It works, but takes too many steps, too long, or too much reading. | Remove steps, set defaults, shorten. |

The kind matters. Building a feature for something that was only hidden makes the product bigger and leaves the friction in place.

### Decide what to fix now

Fix during the run when the point blocks the task, or when the fix is small and inside what may change freely. Leave everything else for the report. A change outside the free area needs the owner's decision. If they decline and the point blocks the task, the run ends as blocked.

### Fix

1. Tell the user to wait. Do not say what is being changed.
2. Tell the owner, without waiting for an answer: the friction point, the verified cause, what changes and where.
3. Make the smallest change that removes this one point. Do not tidy up around it.
4. Fix it where the user was looking, in the product itself. An error message that says what to do next is better than a paragraph in the documentation nobody opened.
5. Work on a branch of its own, one commit per friction point, with a message that names the point and follows the project's conventions. Do not push, open pull requests or bypass checks unless the owner asked for it.

If the owner answers with a restriction, apply it at once, and keep applying it to every later fix of this run.

### Try again

- After a fix for something **missing, broken or heavy**, the same user continues where they stopped.
- After a fix for something **hidden or misleading**, use a fresh user. The first one now knows where to look, so their success proves nothing. The fresh user gets the original briefing and, if needed, one neutral sentence about the starting state ("A project named X already exists"). Describe the situation, never where things are.

A fix is confirmed when a user passes the point without stumbling there. A fix that was not tried again is unconfirmed, and is reported as such.

### Stop

End at the first of these:

- **Done:** a user completed the task, and every fix for a hidden or misleading point was confirmed by a fresh user.
- **Blocked:** a blocking point needs a change that was declined or cannot be made.
- **Budget:** the fix budget is used up. Let the current user finish if they can, and report the rest.

Two failed fixes for the same point end the fixing for that point. The diagnosis is probably wrong; say so.

After the report, stop. Do not start another run on your own.

## The report

Write it in the conversation, not into a file, unless asked otherwise.

- **Outcome:** task, surface, how the run ended, and the last user's verdict.
- **Friction points, the worst first.** For each: where it happened, what the user tried, its kind, its effect, and its state: fixed and confirmed (by the same or a fresh user), fixed but unconfirmed, or open with a suggested fix and the reason it was left.
- **Course of the run:** users, fixes and new attempts in order, with the starting state of every fresh attempt.
- **Decisions:** what you approved, what the owner decided, restrictions that applied, and where you stepped in.
- **What is left behind:** the branch and its commits, and anything real the run created, such as accounts, records or files.

## What goes wrong

- **The observer helps.** One hint and the run measures you, not the product.
- **The user knows too much.** A user briefed with internal names finds everything.
- **Fixing the report instead of the cause.** "Could not find the export" may be a missing permission, not a missing button. Verify first.
- **Documentation as the first resort.** If the user did not read it the first time, more of it will not help.
- **Calling a fix done without another attempt.** Then it is an assumption.
- **Widening the task.** New ideas that come up go into the report, not into this run.
