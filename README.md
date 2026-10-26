# claude-skills

Claude Code skills: `explain`, `dependency-updates`.

| Skill | Purpose |
|---|---|
| `explain` | Technical explanation tailored to the reader's background, gauged by questions first |
| `dependency-updates` | Lists upstream updates and fixes for a project's dependencies with their impact on it, then lets you choose what to adopt. Written for one project: it expects `tools/dependency_watch.py`, a `BACKLOG.md` inbox and a `CHECKPOINT.md`; adapt those paths before reuse |

## Install

```
git clone <this repo> ~/claude-skills
mkdir -p ~/.claude/skills
cp -r ~/claude-skills/explain ~/claude-skills/dependency-updates ~/.claude/skills/
```

Invoke with `/explain <topic>` or `/dependency-updates`. To update later: `git pull`, then copy again.

## License

MIT, see [LICENSE](LICENSE).
