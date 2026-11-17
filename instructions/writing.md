---
description: How I want prose, commits, pull requests and tickets written
---

# Writing

* **Style:** Follow the Google Developer Documentation Style Guide for
  documentation, PR and issue text, and other prose.
* **No em dashes.** Use a comma, colon, parentheses or a new sentence.
* **No AI attribution.** Never mention Claude, Copilot or an AI session in
  commit messages, PRs or issues. Add no `Co-authored-by` trailers and no
  session links or IDs, even if a tool adds them by default.
* **Pull requests:** Keep the description short: what changed and why.
  Don't restate the diff. Add a `Testing` section only if testing went
  beyond CI.
* **Acceptance criteria:** Write them from the user's perspective, as
  observable behavior, preferably as Given/When/Then. Never name
  implementation details such as services, payloads or APIs.
* **Reading issues:** When you read a Jira or GitHub issue, fetch its custom
  fields too.
