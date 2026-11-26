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
* **Pull requests:** Keep the description short, and explain what the
  diff can't: how the change is built and why it was made that way. Don't
  restate the code changes file by file.
* **Frontend changes** ideally come with a screenshot of the result in the
  description: the page or the component. A mock is fine. When there is
  no screenshot, don't mention it or explain why.
* **No routine checks in a pull request.** Leave out test runs and
  validations that belong to the toolchain anyway, such as `npm run lint`
  or `npm run test`. Add a `Testing` section only for testing that went
  beyond that.
* **Acceptance criteria:** Write them from the user's perspective, as
  observable behavior, preferably as Given/When/Then. Never name
  implementation details such as services, payloads or APIs.
* **Reading issues:** When you read a Jira or GitHub issue, fetch its custom
  fields too.
