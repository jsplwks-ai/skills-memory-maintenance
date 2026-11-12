# Claude Code memory layout

- Memory lives in `~/.claude/projects/<project-slug>/memory/`. The current session's memory directory is stated in the system prompt; use that path instead of guessing.
- Each memory is one file holding one fact, with frontmatter `name`, `description`, and `metadata.type` (`user`, `feedback`, `project`, or `reference`).
- `MEMORY.md` in the same directory is the index loaded every session: one pointer line per memory, no memory content.
- Every added, renamed, archived, or deleted memory file needs the matching `MEMORY.md` change in the same approved plan. After a cleanup, check that every index line points to an existing file and every memory file has an index line.
- Memories link to each other with `[[name]]`. When a memory is renamed, merged, or removed, update the links that point to it.
- An archived file must be removed from `MEMORY.md`.
- There is no separate global memory store. Cross-project instructions live in `~/.claude/CLAUDE.md` and rule files, which are out of scope.
