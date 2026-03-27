# Gittodoc Skill

Agent Skill that teaches AI assistants how to use [gittodoc.com](https://gittodoc.com) to convert GitHub repositories into plain-text documentation for context ingestion — no SDK, API key, or MCP required.

## What's in it

| File       | What it covers                                                                                                  |
| ---------- | --------------------------------------------------------------------------------------------------------------- |
| `SKILL.md` | Quick start, step-by-step curl workflow, output format, working with digests, rate limits, macOS compatibility |

The entire workflow fits in a single file — no `references/` directory needed.

## Capabilities

- **Repository-to-text conversion** — turn any public GitHub repo into a single text file containing directory tree and all file contents
- **Token estimation** — each digest includes an approximate token count for context window planning
- **Zero configuration** — no API key, no authentication, no dependencies beyond `curl`
- **AI-optimised output** — flat text format designed for fast indexing by AI coding tools

## Configuration

No configuration required. Gittodoc is a free, unauthenticated web service.

## Example usage

Once the skill is installed, just ask your assistant in natural language:

- "Fetch the gittodoc digest for pallets/flask and summarise its architecture"
- "Ingest the fastapi repo so you can answer questions about its codebase"
- "Use gittodoc to read the source of cyclotruc/gitingest and explain how the ingestion pipeline works"
- "Get the gittodoc output for this repo and find all the API endpoint definitions"
- "How many tokens is the tiangolo/fastapi repo? Check with gittodoc before ingesting it"

The assistant reads the skill, writes the `curl` commands, runs them, and explains the results.

## Installation

### Personal skill (all projects)

Clone into the shared skills directory:

```bash
git clone https://github.com/face0b1101/gittodoc-skill.git \
  ~/.agents/skills/gittodoc
```

Compatible agents (Cursor, Claude Code, VS Code, Codex, and others) scan `~/.agents/skills/` automatically per the [Agent Skills convention](https://agentskills.io/client-implementation/adding-skills-support).

### Project skill (one repo)

From your project root:

```bash
git clone https://github.com/face0b1101/gittodoc-skill.git /tmp/gittodoc-skill
mkdir -p .agents/skills/gittodoc
cp /tmp/gittodoc-skill/SKILL.md .agents/skills/gittodoc/
```

Commit the `.agents/skills/gittodoc/` directory so your team gets it too.

### Claude Desktop

1. Download the zip from the [latest release](https://github.com/face0b1101/gittodoc-skill/releases/latest)
2. In Claude, go to **Settings > Capabilities** and ensure **Code execution** is enabled
3. Under **Skills**, click **Upload skill** and select the zip

Claude will automatically use the skill when your prompt involves gittodoc or repo ingestion. See [Using Skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude) for more detail.

### Verify

After installing, check that your agent can see the skill. In Claude Code, run `/skills` and confirm `gittodoc` appears in the list.

## Why a skill and not MCP?

Gittodoc is a simple web service that takes a GitHub repo path and returns a text file. It doesn't need a protocol translation layer — it needs a clear explanation.

**A skill is just markdown** that tells the LLM how the URL pattern works, gives it `curl` commands for fetching and parsing the HTML response, and documents the output format. The LLM reads this, then writes and executes the actual commands.

**An MCP server** would mean a separate process to run and maintain, tool schemas consuming context tokens on every turn, and version compatibility issues between the server and your environment.

Skills achieve progressive disclosure by design: `SKILL.md` loads when triggered, and the LLM only reads the sections it needs. No tool schema overhead, no server process, zero dependencies.

## Licence

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) — see [LICENCE](LICENCE) for full text.
