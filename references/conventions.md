# Commit & PR Conventions

## Commit messages

Follow Conventional Commits: `<type>(<scope>): <summary>`

| Type | Use |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | No behavior change |
| `style` | Formatting only |
| `chore` | Build / deps / config |
| `test` | Tests |
| `docs` | Docs only |
| `perf` | Performance |

Rules:
- Lowercase, imperative mood, no period at end.
- Max 72 characters in the first line.
- Scope = feature name or layer. Example: `feat(auth): add biometric login`
- Breaking change: add `!` after type/scope. Example: `refactor(api)!: rename response models`
- Never: `fix bug`, `update`, `changes`, `wip` (except on draft PRs).

## PR description template

```markdown
## What
<!-- One sentence: what does this PR do? -->

## Why
<!-- Business or technical reason -->

## Changes
-
-

## Screenshots / recordings
<!-- Attach if UI changed; leave empty otherwise -->

## Checklist
- [ ] flutter analyze clean
- [ ] Tested on iOS
- [ ] Tested on Android
- [ ] No hardcoded strings / magic values
- [ ] No print() statements left
```
