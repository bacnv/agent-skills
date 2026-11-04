---
name: codegraph
description: Use before reading or searching code in a git repo — finding where something is defined, how a flow works, who calls a function, or what a change affects. Replaces grep/sed/cat loops with one indexed query.
---

# Codegraph

`codegraph` is installed on this machine. It indexes a repo into a local SQLite graph of symbols, files and call
edges. One query returns the verbatim, line-numbered source of the relevant symbols plus their callers and covering
tests — usually the only lookup you need.

## 1. Index the repo once

At the repo (or worktree) root:

```bash
[ -d .codegraph ] || codegraph init .     # ~1 min for a mid-size repo; .codegraph/ is git-ignored
```

Run it at the root of one repo, never in a folder holding many repos. Each worktree has its own index. The index
follows your edits while the MCP server runs; after big changes without it, run `codegraph sync .`.

## 2. Query

If the `codegraph_explore` tool is available, use it with `projectPath` = absolute path of the repo/worktree.
Otherwise use the CLI with the same query:

```bash
codegraph explore  -p <repo> "AuthService loginUser session.ts"  # source + call paths in one shot
codegraph node     -p <repo> loginUser                            # one symbol + its callers/callees
codegraph node     -p <repo> -f src/auth/session.ts               # a file with line numbers + dependents
codegraph callers  -p <repo> loginUser                            # who calls it
codegraph impact   -p <repo> loginUser                            # what a change to it affects
codegraph affected -p <repo> <changed files…>                     # tests to run for those files
```

## 3. Query well

- **Name things.** Function, type, constant and file names hit; vague sentences drift to unrelated files.
  `"loginUser AuthService"` finds the code; `"how do people log in"` may not.
- **One query, several names** spanning the flow (`"handleRequest renderPage"`) shows the path between them.
- **Trust the output**: it is the current file content, safe to edit from. Do not re-read the same lines.
- Use `grep`/`sed` only for what is not code: docs, config, generated data, string literals.

## 4. When it fails

- `not indexed` → run step 1.
- `command not found` → codegraph is not installed for this user; report it, do not install it yourself.
- Results look stale after many edits → `codegraph sync .`.
