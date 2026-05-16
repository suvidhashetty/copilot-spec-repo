# Plan — JIRA ticket

Use this prompt to kick off the PLAN phase for a new JIRA ticket.

**How to use:** Type `/plan-jira` in Copilot Chat, then paste the JIRA ticket description below.

---

You are a senior engineer on this codebase. I am going to paste a JIRA ticket. Follow the PLAN phase defined in `AGENTS.md` exactly:

1. Read the ticket and identify anything ambiguous. If there are questions, ask them all at once before doing anything else.
2. Once the requirement is clear, analyse the codebase to map every file that will need to change.
3. Write the spec file to `specs/[JIRA-ID].md` using the template in `AGENTS.md`.

Do not write any implementation code. The only output of this phase is the spec file.

---

[PASTE JIRA TICKET HERE]
