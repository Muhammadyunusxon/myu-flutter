# myu-flutter

A Claude skill containing personal Flutter/Dart coding standards for **Muhammadyunusxon** — Mobile Lead developer (iOS, Android, Flutter Desktop).

Auto-triggers whenever you are writing, reviewing, or discussing Flutter/Dart code. All conventions are extracted from real production projects.

---

## Install

### Option 1 — Download `.skill` file

1. Go to [Releases](https://github.com/Muhammadyunusxon/myu-flutter/releases) and download `myu-flutter.skill`
2. Open **Claude Desktop** → Settings → Skills
3. Drag and drop `myu-flutter.skill` into the skills panel

### Option 2 — Clone and install manually

```bash
git clone https://github.com/Muhammadyunusxon/myu-flutter.git
```

Then copy into your Claude skills folder:

```bash
cp -r myu-flutter ~/.claude/skills/myu-flutter
```

---

## What's inside

```
myu-flutter/
├── SKILL.md                ← Core conventions (always loaded)
└── references/
    ├── bloc.md             ← Bloc: State, Event, handler, BlocBuilder/Listener/Consumer
    ├── infrastructure.md   ← DI (GetIt), HttpService, LocalStorage, Navigation, AppConstants
    ├── testing.md          ← bloc_test + mocktail templates
    ├── ui.md               ← CustomStyle, screenutil, Freezed models, file upload, localization
    └── conventions.md      ← Commit messages + PR description template
```

**SKILL.md** (always in context) covers:

- Developer profile & platform restrictions (iOS / Android / Flutter Desktop only)
- Clean Architecture folder structure
- Code style (Effective Dart, const, trailing commas, no dynamic)
- `ApiResult<T>` error handling pattern
- `AppLogger` setup and usage
- 25 approved packages with purpose
- Workflow: which reference to load per task type

Reference files are loaded on-demand only when relevant to the current task.

---

## Tech stack

| Area | Choice |
|---|---|
| Architecture | Clean Architecture (presentation / application / domain / infrastructure) |
| State management | Bloc + Freezed (new projects), Riverpod + Freezed (legacy) |
| HTTP | Dio with interceptors |
| DI | GetIt |
| Storage | SharedPreferences + FlutterSecureStorage |
| Backend | REST API + Firebase |

---

## Trigger examples

The skill activates automatically on prompts like:

- "Create an auth Bloc"
- "Add a repository for orders"
- "Write a test for this Bloc"
- "Set up GetIt for a new project"
- "Create a data model for User"
- "Write a commit message"
- "Review this Flutter code"

---

## License

MIT
