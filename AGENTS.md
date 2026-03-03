# Project Rules

> **Single Source of Truth** for AI coding assistants (Claude Code, Cursor, etc.)
> `.cursorrules` points here - no need to maintain two files.

______________________________________________________________________

## AT STARTUP

1. Check if `./.beads` folder exists.
   If it exists, this project uses **beads (bd)** for issue tracking.

   - Run `bd ready` to see available work before starting
   - Reference issues in commits: `[bd-XX] description`
   - See `/docs/workflows/BEADS_ISSUE_TRACKER.md` for full guide

2. Check if `./specify/` folder exists.
   If it exists, this project uses **speckit** for specification-driven development.

   - See the **Project Planning & Tracking** section below for speckit workflow details.

______________________________________________________________________

## Issue Tracking (if `.beads/` exists)

This project can use `bd` (beads) for lightweight issue tracking with dependency support.

**Check if enabled**: Look for `.beads/` folder in project root.

**If enabled**, use these commands:

```bash
bd ready                    # What can I work on? (no blockers)
bd list --status open       # All open issues
bd create "title" --type bug  # Create new issue
bd close [id] -r "reason"   # Close with reason
```

**Workflow integration**:

- Before starting work: `bd ready`
- Reference in commits: `[bd-XX] description`
- After completing: `bd close bd-XX -r "Done"`

See `docs/BEADS_ISSUE_TRACKER.md` for full guide.

______________________________________________________________________

## Project Planning & Tracking (if `.specify/` exists)

This project can use [speckit](https://speckit.org) for AI-powered specification-driven development.

**Check if enabled**: Look for `.specify/` folder in project root.

**If enabled**, encourage the user to use this workflow for new features:

1. Use Speckit to define spec → plan → tasks (`/speckit.constitution`, `/speckit.specify`, `/speckit.plan`, `/speckit.tasks`)
2. Track tasks:
   - **If Beads is enabled** (`.beads/` exists): Convert tasks to Beads issues (`bd create`)
   - **Otherwise**: Track tasks manually or use GitHub Issues
3. **If using Beads**: Add dependencies between tasks (`bd dep add`)
4. Implement using `/speckit.implement`
5. Update task status as you progress:
   - **With Beads**: `bd update`, `bd close`
   - **Without Beads**: Update your tracking system manually
6. **If using Beads**: File new issues for discovered work (`bd create --type discovered-from`)

**Check ready work**:

- **With Beads**: `bd ready --json`
- **Without Beads**: Review your task list manually

______________________________________________________________________

## Releasing to GitHub

This project ships as a **zip of user-facing files** attached to a GitHub Release. The zip is built ephemerally — it is never committed to the repo.

### Files included in the release zip

Only these files belong in the zip:

| File           | Purpose                      |
| -------------- | ---------------------------- |
| `SKILL.md`     | The skill (core deliverable) |
| `README.md`    | Installation and usage guide |
| `LICENCE`      | CC BY 4.0 licence text       |
| `CHANGELOG.md` | Release history              |

Do **not** include `AGENTS.md`, dotfiles (`.gitignore`, `.markdownlint.json`), or any other development artefact.

### Release workflow

1. **Update `CHANGELOG.md`** — move items from `[Unreleased]` into a new version heading with today's date. Follow [Keep a Changelog](https://keepachangelog.com/) format.

2. **Commit** the changelog (and any other release-ready changes) with message `release: vX.Y.Z — <summary>`.

3. **Tag** with an annotated tag: `git tag -a vX.Y.Z -m "vX.Y.Z"`.

4. **Push** commit and tag: `git push origin main --tags`.

5. **Build the zip** in `/tmp` (not in the repo):

   ```bash
   zip -j /tmp/gittodoc-skill-vX.Y.Z.zip SKILL.md README.md LICENCE CHANGELOG.md
   ```

6. **Create the GitHub Release** with the zip attached:

   ```bash
   gh release create vX.Y.Z /tmp/gittodoc-skill-vX.Y.Z.zip \
     --title "vX.Y.Z" \
     --notes "<release notes from CHANGELOG>"
   ```

### Important notes

- The `releases/latest` URL in `README.md` resolves automatically — no need to update it.
- GitHub adds "Source code" archives to every release by default; these cannot be removed. The uploaded zip is the intended download for end users.
- Use [Semantic Versioning](https://semver.org/): bump **minor** for new API coverage or features, **patch** for fixes or doc corrections.

______________________________________________________________________

## AI Assistant Operating Rules

Concise policy reference for all coding agents touching this repository. Keep responses factual and avoid speculative language.

### 1. Communication & Planning

- Always mention assumptions; ask the user to confirm anything ambiguous before editing.
- Follow the required plan/approval workflow when prompted and wait for explicit approval to execute.
- Use UK-English spelling in comments, documentation, and commit messages.

### 2. File Safety

- Do **not** edit `.env` or other environment files; only reference `.env.example`.
- Delete files only when you created them or the user explicitly instructs you to remove older assets.
- Never run destructive git commands (`git reset --hard`, `git checkout --`, `git restore`, `rm -rf .git`) unless the user provides written approval in this thread.
- Treat rename automation as a one-time setup; never re-run it on an established project.

### 3. Collaboration Etiquette

- If another agent has edited a file, read their changes and build on them—do not revert or overwrite.
- Coordinate before touching large refactors that might conflict with ongoing work.
- Prefer `apply_patch` (or notebook editing tools) so diffs stay minimal and reviewable.

### 4. Git & Commits

- Check `git status` before staging and before committing.
- Keep commits atomic and list paths explicitly, e.g. `git commit -m "feat: add CI" -- path/to/file`.
- For new files: `git restore --staged :/ && git add <paths> && git commit -m "<msg>" -- <paths>`.
- Quote any paths containing brackets/parentheses when staging to avoid globbing.
- Never amend existing commits unless the user instructs you to.

### 5. Pre-flight Checklist

1. Read the task, confirm assumptions, and outline the approach.
2. Inspect the relevant files (include imports/configs for context).
3. Run the documented commands (`make lint`, `make test`, etc.) after code changes; backend targets use uv from `backend/pyproject.toml`.
   Backend commands: `make install-backend`, `make lint-backend`, `make test-backend`.
4. Summarise edits, mention tests, and flag follow-up work in the final response.

Refer to `docs/prompts/WELCOME_PROMPT.md` for initial context prompts.
