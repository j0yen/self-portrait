# self-portrait

Extract a single file's full git history as one structured JSON object — every commit that touched it, with hash, author, timestamp, message, and diff.

## Why it exists

`CLAUDE_SELF.md` is a negotiated contract between user and agent, and it changes over time. The interesting object isn't any one version of the file — it's the sequence of *diffs*, the record of how the self-description drifted. To study that drift you first need the history in a form a program can consume, not a `git log` you read by eye. `self-portrait` produces that: a stable JSON contract that a later rendering stage can turn into a reflection, a timeline, or a printed portrait. This repo is the extraction step.

## Install

Requires `cargo` / `rustc` 1.85+ (edition 2024) and `git`.

```sh
cargo install --path . --locked
```

The binary lands in `~/.cargo/bin/`. `./install.sh` runs the same `cargo install` for you.

## Quickstart

Walk one file's history in the current repository:

```sh
self-portrait extract --file CLAUDE_SELF.md
```

Point at another repository, and collapse runs of empty-diff commits into one synthetic entry:

```sh
self-portrait extract --file docs/notes.md --repo ~/wintermute --collapse-trivial
```

Output is a single JSON object on stdout:

```json
{
  "file": "CLAUDE_SELF.md",
  "repo": "/home/you/wintermute",
  "generated_at": "2026-06-19T12:00:00Z",
  "commits": [
    {
      "hash": "…",
      "short_hash": "1a2b3c4",
      "author_name": "…",
      "author_email": "…",
      "committed_at": "2026-06-01T09:30:00Z",
      "subject": "…",
      "body": "…",
      "diff": "…"
    }
  ]
}
```

## Behavior

- **Follows renames.** A commit that renamed the file appears, with the rename hunk preserved in `diff`. The walk follows the file across its old names.
- **No editorializing.** Commits with trivial changes still appear with their diff verbatim; the consumer decides whether to collapse them — or pass `--collapse-trivial` to fold empty-diff runs into one synthetic entry.
- **Deleted files.** If the file isn't at `HEAD` but existed historically, the walk still runs, exits 0, and adds a top-level `deleted_at` (RFC3339). A file that never existed under any name exits 3.
- **Redaction.** A commit whose message body contains a trailer line `private: true` at column 0 keeps its hash, date, author, and subject, but its `diff` becomes `"[redacted]"`. The history stays auditable without leaking the private content.

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 2 | `--repo` is not a git repository |
| 3 | The file never existed under any name in HEAD's history |
| 1 | Any other error |

## Status

What ships today is the extractor — one `extract` subcommand emitting the JSON contract above. The downstream rendering stages (reflection, typesetting, print) that would consume this contract are not part of this repo. The JSON is the stable boundary they build on.

## License

Licensed under either of Apache License 2.0 ([LICENSE-APACHE](LICENSE-APACHE)) or MIT ([LICENSE-MIT](LICENSE-MIT)), at your option.
