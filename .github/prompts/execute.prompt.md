# Execute

Use this prompt to begin the EXECUTE phase once the spec is approved.

**How to use:** Type `/execute` in Copilot Chat with the spec file open or referenced.

---

The spec at `specs/[JIRA-ID].md` is approved. Begin implementation.

Follow the EXECUTE phase defined in `AGENTS.md` exactly:

1. Re-read the spec file in full before writing any code.
2. Load all relevant skill files for the file types you will be creating.
3. Work through the implementation checklist in order — do not skip steps.
4. Tick off each checklist item in the spec file as you complete it.
5. Run tests after each logical unit of work and fix failures before continuing.
6. If you encounter anything not covered by the spec, stop and ask before continuing.

When complete, update the spec status to `Implemented` and summarise what was built.
