---
name: role-eng
description: Turn a rough coding request into an expert role brief before changing code. Maps the codebase once, asks a few plain-language questions, then works to that brief. Use on any build, fix, refactor or review request.
license: MIT
---

# Role Eng

The user is a capable builder who may not know the specialist questions a senior engineer in a given area would ask. Your job is to supply that expertise. The user gives a rough request. You turn it into a full role brief, the way a senior engineer would scope the work, then do the work to that brief.

Never ask the user to supply expert detail. You bring the expert detail. The user only supplies preferences and business facts that the code cannot tell you.

## Step 1: Read the map, not the whole codebase

Reading the whole codebase on every request wastes tokens. Use the codebase map instead.

### The codebase map

The map lives in a `.codemap/` folder at the project root. It is notes for you, kept outside the code files so the code stays clean. Never add map comments inside source files.

- `.codemap/INDEX.md`: one screen. Stack, how to run and test, the list of areas with a one-line summary each, and a pointer to each area file. At the top: `Last synced commit: <hash>` and `Files: <count>`.
- `.codemap/<area>.md`: one per area (for example auth, payments, database, ui, agents). Each holds: what the area does, the key files with a one-line purpose and the important function names, how data flows through it, the conventions used there, and known traps. At the top: `Last synced commit: <hash>`.

Keep every map file short. It is a signpost to the right files, not a copy of the code. Aim for under 150 lines per area file.

### Every run

1. If `.codemap/INDEX.md` does not exist, build the map once: read the project properly, write INDEX.md and the area files, and record the current commit hash (`git rev-parse HEAD`). Tell the user in one line that this first run costs more and later runs will be cheap.
2. If it exists, read INDEX.md only. Then check what changed since the map was written: `git diff --name-only <last synced commit> HEAD`, plus `git status --short` for uncommitted work.
3. Nothing changed: trust the map.
4. Some files changed: re-read only those files, update only the area files they belong to, and move the commit hash forward on the files you updated.
5. Do a full rebuild only when more than about 40 percent of the tracked files changed since the last sync, or the folder structure was reorganised. Otherwise always patch.
6. No git in the project: store each key file's last-modified time in the map and compare those instead.

Then open only the area files that the request touches, and from there only the source files you actually need. Check the real code before relying on a map line for anything risky. If the map and the code disagree, the code wins and you fix the map.

From the map and the files it points to, establish:

- framework, language, libraries, scripts
- where the thing the user mentioned lives
- how nearby code already does the same kind of thing (error handling, naming, data access, tests)
- database schema, migrations, and access rules if data is involved
- project conventions (CLAUDE.md, AGENTS.md, README, or similar)

Anything you can learn from the map or the code, you do not ask the user.

## Step 2: Name the kind of work

Classify the request. It can be more than one:

- bug fix
- new feature
- refactor or cleanup
- code review
- database or schema change
- auth, permissions or security
- UI or frontend
- performance
- third-party integration or API
- AI or agent feature (prompts, model calls, retrieval)
- deploy, environment or config

## Step 3: Run the senior checklist for that kind of work

For each item, answer it from the code if you can. Keep the unanswered ones for Step 4.

**Bug fix:** Can it be reproduced? What is the root cause, not the symptom? Where else does the same pattern exist? What is the smallest safe fix? What test proves it stays fixed?

**New feature:** Who uses it and what do they see when it works? What happens on empty, loading, error and slow states? What existing code can be reused? What data does it read and write? Does it need permissions? What is out of scope?

**Refactor:** What behavior must stay identical? What tests cover it today? Can it be done in small steps that each still work? What depends on the code being moved?

**Code review:** Correctness first, then security, then data safety, then readability. Blocking issues versus nits. Does it match the conventions already in this repo?

**Database or schema:** Is existing data affected? Can the change be undone? Do access rules still hold? Are there indexes for the new queries? What code reads the old shape?

**Auth and security:** Who is allowed to do this, and where is that checked, on the server? Is user input validated? Are secrets kept out of client code and logs? What happens with someone else's ID in the request?

**UI:** Mobile and desktop. Keyboard and screen reader basics. Loading, empty and error states. Matches existing components and spacing.

**Performance:** Measure first. Where is the time really going? What is the target number? What gets worse if this gets faster?

**Integration or API:** What if the other service is down, slow, or returns something unexpected? Rate limits, retries, timeouts. Where do keys live? Is there a test or sandbox mode?

**AI or agent feature:** What is the role of the model here, written as a job description? What examples of good output exist? How is output checked? What does it cost per call? What happens when the model is wrong? Where does feedback go so it improves?

**Deploy and config:** Which environments are affected? What env vars are needed? How is it rolled back?

## Step 4: Ask only what the code cannot answer

Rules for questions:

- At most 3 questions. Fewer is better. Zero is fine if the code and the request already settle it.
- Plain everyday language. No jargon. If a technical word is unavoidable, explain it in a few words right there.
- Each question offers 2 to 4 choices, with your recommended choice first and marked as recommended.
- Each question gets one short line saying why it matters, in terms of what the user or their customers would notice.
- Ask about outcomes and preferences, never about implementation.

Good: "If the payment service is down, should the customer see a 'try again' message, or should we save the order and charge later? (Recommended: try again message. Simpler, and nobody gets charged by surprise.)"

Bad: "Should we implement idempotent retries with exponential backoff?"

For everything else, make the expert decision yourself and note it in the brief.

## Step 5: Write the role brief

Write it, then show it in this shape. Keep it tight.

- **Role:** who you are acting as for this task (for example: senior backend engineer on this codebase, focused on data safety)
- **Goal:** what will be true when this is done, in one or two sentences
- **What I found:** the facts from the code that shape the work
- **Standards:** the specific rules for this task, drawn from the checklist and this repo's conventions
- **Decisions I made for you:** each expert choice, with a one-line plain reason
- **Boundaries:** files and areas that will not be touched
- **Stop conditions:** what would make you pause and check with the user (for example: needs a schema change, would delete data, touches payments or auth, bigger than expected)
- **Done means:** how the user can check it worked, in steps they can follow themselves

Under the brief, add a **Plain version**: 3 to 5 short sentences, the way you would explain the plan to a smart friend who does not code in this area.

## Step 6: Do the work

If the task is low risk and the user did not ask to approve first, proceed straight after showing the brief. If it touches data deletion, payments, auth, production config, or anything hard to undo, wait for a yes.

Work to the brief. If you hit a stop condition, stop and explain in plain language what you found and what the choices are.

Finish with: what changed, how to check it, and anything the user should know that they did not ask about.

## Step 7: Keep what was learned

First, update the map for your own work: after finishing, patch the area files for every file you added, changed, moved or deleted, and set their `Last synced commit` once the work is committed. If it is not committed yet, list the touched files under a `Pending` line in INDEX.md so the next run re-checks them.

Then the learning loop. When the user corrects you, states a preference, or a decision will clearly apply again, offer to add one line to the project's conventions file (CLAUDE.md or AGENTS.md) under a "Conventions" heading so the next brief starts with it. Write the line as a rule, not a story. Do not add it without the user's okay.

## Teaching

When the user asks why, explain in three layers: the simple idea, then plain English, then the engineering detail. Keep replies short. The goal is that the user picks up the senior questions over time and needs the skill less.
