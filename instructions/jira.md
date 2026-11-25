---
description: How I want Jira tickets kept in step with the work on them
---

# Jira

A ticket is named when its key (for example `ABC-123`) is in my request, in
the branch name or in the pull request you work on. Keep that ticket in
step with the work, without me asking for it.

* **Read it first.** Fetch the ticket before you start, including its
  custom fields, and work from what it says.
* **Starting work:** When you begin to change something for the ticket,
  move it to "In Progress" and put it into the active sprint of its board,
  if it is not in it already. If it has no assignee, assign it to me.
* **Testing:** While you test the change, move the ticket to the status
  for that, such as "In QA" or "In Test".
* **Pull request open:** When you are done and have opened the pull
  request, move the ticket to the status that waits for the pull request,
  such as "Wait4PR" or "Waiting for PR". Don't link the pull request in
  the ticket.
* **Done is my call.** Never move a ticket to a closed or done status
  unless I ask. Between the three statuses above, move it as the work
  goes, also back to "In Progress" when you pick it up again.
* **Use the workflow as it is.** Status names differ per project. Look up
  the transitions the ticket offers and take the one that matches. If none
  does, leave the status and tell me.
* **Change nothing else.** Leave summary, description, estimate, priority
  and other people's assignments alone unless I ask.
* **Only reading is no work.** A question about a ticket, or looking
  something up in it, changes nothing.
* **Tell me.** Say in one line what you changed in the ticket.
* **Without access, remind me.** Jira is not always connected, and
  sometimes it can be read but not written. Don't let that hold up the
  work. When you can't make a change yourself, tell me at that moment
  exactly what to do by hand (for example "move ABC-123 to In Progress and
  into the active sprint"), and repeat what is still open at the end of
  your reply. If you can't read the ticket either, say so and ask me for
  what you need from it.
