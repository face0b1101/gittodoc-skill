---
name: gittodoc
description: >-
  Convert GitHub repositories into plain-text documentation via gittodoc.com
  for AI context ingestion. Use when needing to read, understand, or ingest an
  entire GitHub repository's source code, or when the user mentions gittodoc,
  gitingest, or wants to feed a repo to an AI tool. No authentication required.
---

# Gittodoc

Gittodoc converts any public GitHub repository into a single plain-text file containing the directory tree and all file contents. No authentication, no SDK, no dependencies — just `curl`.

Website: <https://gittodoc.com>

## Quick Start

Generate and fetch a repository digest in one command:

```bash
S3_URL=$(curl -sL "https://gittodoc.com/owner/repo" | grep -oP 'https://gitdocs1\.s3\.amazonaws\.com/digests/[^"]+\.txt')
curl -s "$S3_URL"
```

Replace `owner/repo` with the GitHub owner and repository name (e.g. `pallets/flask`).

## Step-by-Step

### 1. Trigger Digest Generation

Send a GET request to `https://gittodoc.com/{owner}/{repo}`. This clones the repository (shallow, single-branch) and processes it into a text digest.

```bash
curl -sL "https://gittodoc.com/pallets/flask" -o /tmp/gittodoc-response.html
```

The response is HTML. Processing time depends on repository size — small repos return in seconds, large repos may take longer.

### 2. Extract the S3 URL

The HTML response contains a link to the generated `.txt` file on S3:

```bash
S3_URL=$(grep -oP 'https://gitdocs1\.s3\.amazonaws\.com/digests/[^"]+\.txt' /tmp/gittodoc-response.html)
echo "$S3_URL"
```

If `S3_URL` is empty, the repository may not exist, may be private, or processing may have failed. Check the HTML for an error message:

```bash
grep -oP 'Error[^<]+' /tmp/gittodoc-response.html
```

### 3. Download the Text Digest

```bash
curl -s "$S3_URL" -o /tmp/repo-digest.txt
```

### Combined One-Liner

```bash
curl -sL "https://gittodoc.com/pallets/flask" \
  | grep -oP 'https://gitdocs1\.s3\.amazonaws\.com/digests/[^"]+\.txt' \
  | xargs curl -s
```

### macOS Compatibility

The `-P` flag for grep (Perl regex) is not available on macOS by default. Use `grep -oE` instead:

```bash
S3_URL=$(curl -sL "https://gittodoc.com/pallets/flask" \
  | grep -oE 'https://gitdocs1\.s3\.amazonaws\.com/digests/[^"]+\.txt')
curl -s "$S3_URL"
```

## Output Format

The text digest follows this structure:

```
Repository: owner/repo
Files analyzed: 42

Estimated tokens: 15.2k

Directory structure:
└── owner-repo/
    ├── README.md
    ├── src/
    │   ├── main.py
    │   └── utils.py
    └── tests/
        └── test_main.py

================================================
File: owner-repo/README.md
================================================
<file contents>

================================================
File: owner-repo/src/main.py
================================================
<file contents>

...
```

Key fields:
- **Repository** — `owner/repo` identifier
- **Files analyzed** — total number of files processed
- **Estimated tokens** — approximate token count (tiktoken-based), useful for context window planning
- **Directory structure** — full tree view of the repository
- **File contents** — each file separated by `================================================` delimiters

## Working with the Output

### Save to a Local File

```bash
curl -s "$S3_URL" > /tmp/flask-digest.txt
wc -l /tmp/flask-digest.txt
```

### Search Within the Digest

```bash
curl -s "$S3_URL" | grep -n "def create_app"
```

### Extract a Single File's Contents

```bash
curl -s "$S3_URL" \
  | sed -n '/^File: .*README\.md$/,/^={48}$/p'
```

### Check Token Count Before Ingestion

```bash
curl -s "$S3_URL" | head -5
```

The `Estimated tokens` line in the header helps decide whether the digest fits in your context window.

### Pipe Directly into Context

When using the digest as AI context, you can fetch and pipe it inline:

```bash
DIGEST=$(curl -sL "https://gittodoc.com/owner/repo" \
  | grep -oE 'https://gitdocs1\.s3\.amazonaws\.com/digests/[^"]+\.txt' \
  | xargs curl -s)
echo "$DIGEST" | head -5
```

## URL Patterns

| URL | Behaviour |
| --- | --------- |
| `gittodoc.com/{owner}/{repo}` | Generate digest for the default branch |
| `gittodoc.com/github.com/{owner}/{repo}` | Redirects to `/{owner}/{repo}` |

Only public GitHub repositories are supported. Private repositories return a "Repository not found" error.

## Rate Limits & Caveats

- **Rate limit:** 10 requests per minute per IP address.
- **Public repos only** — private repositories are not supported (the server clones via unauthenticated HTTPS).
- **Large repos may timeout** — repositories with many files or very large files may fail to process within the server timeout.
- **S3 links are temporary** — generated digests are periodically cleaned up. Re-generate if a link expires.
- **File size filtering** — the server applies a default max file size filter (approximately 50 KB per file). Very large individual files may be excluded.
- **No formal API** — the API is marked as "under development" at `gittodoc.com/api`. The HTML-scraping approach documented here works reliably but may change.

## Tips

- **Check token count first** — read the first 5 lines of the digest to see `Estimated tokens` before loading the full content into a context window.
- **Prefer focused repos** — smaller, well-structured repos produce the most useful digests. For monorepos, consider if a subdirectory would be more useful.
- **Cache locally** — save digests to `/tmp/` to avoid re-generating on repeated use.
- **Combine with search** — pipe the digest through `grep` to find specific functions, classes, or patterns rather than reading the entire file.
- **Use `grep -oE`** on macOS — the Perl regex flag (`-P`) requires GNU grep. The extended regex flag (`-E`) works natively on macOS.
- **Mind the rate limit** — if you need to process multiple repos in sequence, add a short delay between requests.
