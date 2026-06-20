# claude-code-security-audit

A ready-to-use application-security (AppSec) guardrail for **Claude Code**, provided
per stack. Each bundle adds an **`/audit`** slash command to your project that runs a
white-box security review of your pending changes (`git diff`) and returns a structured
report - plus a "passive guardrail" mode that secures code as it is generated.

> These are Claude Code **slash commands** (`.claude/commands/audit.md`) plus a project
> `CLAUDE.md`, **not** Claude "skills".

Available in two languages: **`en/`** (English) and **`fr/`** (French). The language of
the bundle files drives the language of the audit report.

## Stacks

| Stack                              | Path                                                  |
| ---------------------------------- | ----------------------------------------------------- |
| PHP (PDO/mysqli, Laravel, Symfony) | `<lang>/php/claude-code-audit-php/`                   |
| Node.js / Express                  | `<lang>/node-express/claude-code-audit-node-express/` |
| Next.js                            | `<lang>/nextjs/claude-code-audit-nextjs/`             |
| Django / DRF                       | `<lang>/django/claude-code-audit-django/`             |
| Flask                              | `<lang>/flask/claude-code-audit-flask/`               |
| Ruby on Rails                      | `<lang>/rails/claude-code-audit-rails/`               |
| Java / Spring Boot                 | `<lang>/spring/claude-code-audit-spring/`             |

## Install

Copy the bundle for your stack (the `.claude/` folder **and** `CLAUDE.md`) to the
**root of your project** - where you run `claude` and where your git repo lives.

```bash
# example for a PHP project, from the project root
cp -r path/to/en/php/claude-code-audit-php/.claude .
cp    path/to/en/php/claude-code-audit-php/CLAUDE.md .
```

If you already have a `.claude/` or `CLAUDE.md`, **merge** instead of overwriting: put
`audit.md` in `.claude/commands/`, the other two `.md` files in `.claude/`, and copy the
"security" block into your existing `CLAUDE.md`.

Then start a **new** Claude Code session (commands are detected at startup) and check
with `/help` that `audit` appears.

## Usage

```
/audit                          # audit pending changes (git diff)
/audit src/Controller/Foo.php   # audit a specific file
```

The report sorts findings by severity (🔴 critical / 🟠 major / 🟡 minor / ℹ️ info),
gives the risk, the vulnerable code and a fix for each, and ends with `🟢 All clear`
or `🔴 N vulnerability(ies)`.

## Bundle contents

```
claude-code-audit-<stack>/
├── CLAUDE.md                          # persona + passive guardrail
└── .claude/
    ├── commands/audit.md              # the /audit command
    ├── SECURITY_RULES.md              # vulnerability checklist (per stack)
    └── security_report_template.md    # report format
```

Feel free to adapt `SECURITY_RULES.md` to your business logic and framework version.

## Limitations

This is a **review copilot**, not a static analyzer (SAST) or a pentest. An LLM can miss
a vulnerability or flag a false positive: keep a critical eye on the report and keep your
other safeguards (tests, SAST, human review). Only use this kit on code you are
responsible for.

## License

MIT (suggested - adjust to your choice).

---

🇫🇷 Version française : [README.fr.md](README.fr.md)
