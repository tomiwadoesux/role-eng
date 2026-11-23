# role-eng

Role engineering for your codebase. Give your coding agent a rough request, and it scopes the job the way a senior engineer would before it touches any code.

## The problem

A good prompt needs expert detail: what to check, what to leave alone, when to stop. If you are not a specialist in that part of the code, you do not know what to ask for. So the agent guesses.

## What role-eng does

1. **Maps your codebase once.** It writes short notes into a `.codemap/` folder, outside your source files. Later runs read the map and use git to re-read only the files that changed, which keeps token use low.
2. **Runs the senior checklist** for the kind of work: bug fix, feature, refactor, review, database, security, UI, performance, integration, AI feature, or deploy.
3. **Asks you at most three questions, in plain language**, each with choices and a recommended pick. No jargon.
4. **Writes a role brief**: role, goal, standards, boundaries, stop conditions, and how you can check the result. A plain version sits underneath.
5. **Does the work to that brief**, pausing first on anything risky such as payments, auth, or deleting data.
6. **Learns.** It updates the map after its own changes and offers to save your corrections to the project's conventions file.

## Install

```bash
npx skills add tomiwadoesux/role-eng
```

Works with any agent that supports the Agent Skills format, including Claude Code, Cursor, and Codex.

## Use

Ask for what you want in your own words:

```
/role-eng add a way for riders to mark a delivery as failed
```

Example question it might ask:

> If a rider marks a delivery as failed by mistake, should they be able to undo it themselves, or should only an admin undo it? (Recommended: rider can undo within 5 minutes. Fewer support requests.)

## Notes

- The first run on a project costs more, because it builds the map. Add `.codemap/` to git so your whole team shares it.
- If the map and the code ever disagree, the agent trusts the code and fixes the map.

## Author

Built by Simeon Ayano at [SgOps](https://github.com/Simeon-gab).

## License

MIT
