# Syncing this fork

This repo follows [`cursor/plugins/pstack`](https://github.com/cursor/plugins/tree/main/pstack) through the mirror at `origin/main`, and adds two things of its own:

- **Mirror edits**: the top of `README.md`, harness-neutral rewrites in some skills, and kebab-case skill names.
- **A Chinese digest layer**: `_digest/`, `_faq_on_digest/`, and `_handbook_for_agent_engineering/`.

## Branches

| Branch | Holds |
|---|---|
| `origin/main` | The synced mirror: Cursor's files plus the mirror edits |
| `main` | This clone's sync line, fast-forwarded to `origin/main` |
| `ethan` | `main` plus the digest layer |

Never copy Cursor's files onto `main` or `ethan` directly. Sync by merging, so Git applies only what changed upstream and keeps the local edits.

## Sync steps

1. Update the sync line:

   ```bash
   git switch main
   git pull --ff-only origin main
   ```

2. See what changed:

   ```bash
   git log --oneline <last-synced-short-sha>..main
   git diff --stat <last-synced-short-sha>..main
   ```

3. Merge it into the digest branch:

   ```bash
   git switch ethan
   git merge main
   ```

4. Write a new record in [`_digest/_change_log/`](./_digest/_change_log/README.md), following its template and impact matrices.

5. Update the digest pages the record marks as affected. Check the indexes and counts (`54` skills, `23` playbooks, `24` principles at the 0.15.25 baseline), and check that the dependency table in `README.md` still matches `_digest/system/distribution.md`.

6. Refresh the bundled Comment Sicko prompt, then check that no new Cursor-only instructions arrived:

   ```bash
   cp agents/comment-sicko.md skills/no-comments/references/comment-sicko.md
   git diff <last-synced-short-sha>..main -- skills | grep -nE '\.cursor/|agent-transcripts|cursor-team-kit|create-skill|Task'
   ```

   Rewrite any new hits the same way as the existing edits. The **Harness** section in `skills/poteto-mode/SKILL.md` lists the mappings.

7. Commit and push:

   ```bash
   git push origin ethan
   ```

   Push `main` too if you changed it.

## Conflict policy

- **`main` against `origin/main`**: keep Cursor's new meaning, then reapply the harness-neutral wording.
- **`ethan` against `main` in a digest file**: keep the new source behavior, then rewrite the explanation against it. If the conflict is in `skills/`, the merge touched mirror edits; treat it like the first case.

The digest layer describes upstream behavior. It must never become a second copy of it.

## Digest layer

- [`_digest/`](./_digest/README.md) — system, patterns, principles, playbooks, and per-skill explanations.
- [`_faq_on_digest/`](./_faq_on_digest/README.md) — question-indexed decisions.
- [`_handbook_for_agent_engineering/`](./_handbook_for_agent_engineering/README.md) — a standalone method handbook (skeleton).

The page contracts, writing rules, and invariants are in [`_digest/MAINTAINING.md`](./_digest/MAINTAINING.md).
