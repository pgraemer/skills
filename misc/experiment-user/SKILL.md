---
name: experiment-user
description: Run a usability experiment where a simulated user performs a task through one channel of a product, for example an MCP server or the web app in a browser. This session observes, has friction fixed mid-run by a builder, and continues or replays until the task works. Use when the user invokes /experiment-user, e.g. "/experiment-user through mcp <task>" or "/experiment-user through the ui <task>, up to 5 fixes". Ends with a friction report in the chat.
---

# Experiment: a simulated user tries a task

A simulated user tries one task through one channel. The observer verifies what the user reports, has a builder fix friction during the run, and lets the same or a fresh user try again. The run produces a friction report in the chat.

This skill is project-independent. Everything about the product under test comes from the project profile (section 0).

## Roles

- **Operator:** the person who invoked the skill. Decides everything expensive, irreversible, or outside the builder's free zone, and may constrain the builder at any time.
- **Observer** (this session): sets up the run, spawns the user and the builder, approves cheap user actions, verifies claims, and writes the report. It never does the user's task itself.
- **User** (separate agent): the persona from the profile. It tries the task through one channel and reports what it couldn't do.
- **Builder** (separate agent, spawned on demand): fixes friction in the code while the user is paused.

## Harness mapping

The skill describes actions, not tools. Use the equivalents of the harness you run in.

| Action | Claude Code | Session-based harness |
|---|---|---|
| Spawn an agent | `Agent`, in the background | `create_session` with `session_kind: "chat"`, `kickoff.mode: "interactive"`, `coordinate_with_creator: true`, `notify_on_idle: "always"` |
| Continue an agent | `SendMessage` | message the session |
| Ask the operator (blocking) | `AskUserQuestion` | `ask_user` |
| Tell the operator (non-blocking) | a normal chat message, then keep working | same |
| Scratch folder | the session scratchpad | the session's `files/` folder |

Spawn agents so that tool calls to the channel aren't auto-denied (interactive, not autopilot). If the harness can't run a second agent, stop and tell the operator.

## 0. Load the project profile

Read `.agents/experiment-user.md` in the project root. It holds, as prose under these headings:

- **Channels:** each channel's name, type (tool or browser, or custom rules), trigger phrases, how to reach it, how sign-in works, and known quirks.
- **Persona:** who the user is and what they know and don't know.
- **Preconditions:** how to start the product and how to tell that it's up.
- **Verification sources:** where the observer checks claims and job state (code, database, logs, cloud access), with the commands.
- **Builder zones:** which paths the builder may change freely per channel, and which need the operator's approval.
- **Expensive actions:** what counts as costly, slow, or irreversible in this product.

If the file is missing, build it once: discover what you can from the repository (start commands, MCP configuration, URLs), ask the operator for the rest, and write the file. During any run, append quirks you learn to the matching heading.

## 1. Parse the invocation

Read three things from the operator's text:

- **Channel:** match against the profile's trigger phrases. Exactly one channel per run. Comparing channels means two runs with the same task.
- **Task:** the rest of the text, verbatim.
- **Fix budget:** "up to N fixes" if given, otherwise 3.

If the channel or the task is missing or unclear, ask the operator before spawning anything, offering the profile's channels as choices.

## 2. Preconditions

- Check the profile's preconditions for the product and the chosen channel. Ask the operator to fix whatever is missing.
- Create the branch `experiment/<slug>` in the checkout that the running product is served from, so that fixes reach the user. If that checkout has uncommitted changes or is on the default branch, tell the operator and branch from there. Never commit to the default branch.
- If the task needs input files, ask the operator for a local path.

## 3. Spawn the user

Name the agent "Experiment user". Build its kickoff prompt from the persona, the shared rules, and the channel rules. Don't give hints, IDs, names, or in-app locations beyond the operator's task, unless the operator says so.

Shared:

> <Persona from the profile>
> Task: <the operator's task, verbatim>
> Rules:
> - Before any action that creates, changes, deletes, starts, or uploads something, send a proposal with your reasoning and wait for approval.
> - If you get stuck, say what you tried, where you looked, and what you needed. Don't guess identifiers and don't work around missing functionality.
> - After every step, report what you did and what you saw.
> - Report every point of friction: anything missing, confusing, hidden, wrong, slow, too large, poorly described, or without feedback.
> - When you're told to wait, wait. When you're told to try again, try again as you normally would.
> - When you're done or blocked, give a verdict on the task and list the friction points in priority order.

Tool channel (an MCP server or a CLI):

> - Use only <the channel's tools>. Don't use code, the database, other APIs, or a browser.
> - List the calls you made in each report.

Browser channel:

> - The app is at <URL from the profile>. If you see a login page, stop and ask for a sign-in.
> - Use only the browser tools: navigate, snapshot, screenshot, click, type, and upload.
> - Find your way like a person would: menus, page titles, and buttons. Don't type URLs you haven't seen in the app.
> - Don't use code, the database, or APIs.
> - Name the pages you used in each report, and take screenshots of friction.

For any other channel type, use the rules written in the profile. Add channel-specific restrictions from the profile to either template.

## 4. Observer loop

### Proposals

- **Approve yourself:** actions that are local, cheap, reversible, and within the task. Approve harmless mistakes too: a dead end is a finding. Log each approval.
- **Ask the operator:** anything the profile lists as expensive, plus deletes, actions on shared or non-local resources, and anything outside the task. Add your own judgment (risks, likely mistakes) and relay the decision verbatim.

### Claims

Before a claim becomes a fix or a finding, verify it against the profile's verification sources, and classify it:

- **Missing:** the capability doesn't exist.
- **Hidden:** it exists, but the user couldn't find it.
- **Broken:** it exists and fails.
- **Misled:** the product's output caused a wrong conclusion, for example a truncated list read as complete.

Don't correct the user. A real user has nobody to correct them, so let the wrong conclusion stand and record where it leads. Intervene only when the wrong conclusion is about to drive an expensive or irreversible action, or when the user loops without producing new information. Log each intervention with a time and a reason.

### Fixes

Fix mid-run when the friction point is a blocker, or when the fix is cheap and inside the builder's free zone. Defer everything else to the report.

For each fix:

1. Tell the user to wait. Never tell it what is being changed.
2. Send the operator the plan as a non-blocking message: the friction point, the verified cause, what will change and where, and whether a restart is needed. Don't wait for an answer.
3. Spawn or continue the "Experiment builder" with the same plan, the zone rules, and all standing constraints.
4. When the builder is done, send the operator the result: files changed, the commit, and any deviation from the plan.

Rules for the builder:

- It may change the free zone of the current channel without approval.
- A change outside the free zone needs the operator's approval. Ask, and say why the free zone can't solve it. If the operator declines and the friction point is a blocker, the run ends as blocked.
- It works in the checkout the product runs from, on `experiment/<slug>`, with one commit per fix whose message names the friction point and follows the project's commit conventions.
- It doesn't push, open pull requests, or bypass commit hooks.
- It doesn't run tests, type checks, or linters during the run beyond what the commit hooks enforce.
- It reports when a change needs a restart, and the observer handles the consequences listed in the profile (for example, a dropped sign-in).

When the operator replies with a constraint, forward it to the builder at once, have it adjust or revert, record the constraint in the log, and include it in every later builder instruction of this run.

### Continue or replay

- **After a fix for something missing or broken:** tell the same user to try again. It continues where it stopped.
- **After a fix for something hidden or misleading:** replay with a fresh user. It gets the original kickoff prompt plus one neutral sentence about the starting state, for example "A dataset named X already exists." Describe the situation, never where things are. The fresh user starts at the fixed point, not from scratch. Replay from scratch only when the fixed point is at the very start.

A fix is confirmed when the user gets past the fixed point without stumbling there.

### Waiting

For long-running jobs, check their state through the profile's verification sources. Tell the user to wait and resume it when the job finishes. Don't poll in a tight loop.

### Log

Keep a timestamped log and any screenshots in the scratch folder. They're working material, not a deliverable.

## 5. End conditions

End the run at the first of these:

- **Success:** a user completed the task, and every fix for a hidden or misleading point was confirmed by a fresh user.
- **Blocked:** a blocker needs a change the operator declined, or one the builder can't make.
- **Budget:** the fix budget is used up. Stop fixing, let the current user finish if it can, and report the rest.

Two failed fixes for the same friction point end fixing for that point: mark it deferred and say that the diagnosis is probably wrong.

After the report, stop. Don't start another experiment, push, or open pull requests.

## 6. Report

Write the report as a chat message. Don't deliver it as a file.

- **Outcome:** channel, task, end condition, and the final user's verdict.
- **Friction points in priority order.** For each one:
  - where it happened (tool or page)
  - what the user tried
  - its class: missing, hidden, broken, or misled
  - the impact
  - its status: fixed and confirmed (commit, and whether the same or a fresh user confirmed it), fixed but unconfirmed, or deferred (suggested fix, and why it wasn't done during the run)
- **Timeline of rounds:** each user, fix, continuation, and replay in order, with the starting state of every replay.
- **Decisions:** proposals the observer approved, the operator's decisions, standing builder constraints, and interventions with their reasons.
- **Leftovers:** real artifacts created, with identifiers, and the experiment branch with its commits.
