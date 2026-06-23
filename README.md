# claude-code-security-audit

A ready-to-use application-security (AppSec) plugin for **Claude Code**. It adds an
**`/audit`** slash command that **auto-detects your stack**, applies the matching
vulnerability checklist, runs a white-box security review of your pending changes
(`git diff`), and returns a structured report.

Supported stacks: **PHP**, **Node.js / Express**, **Next.js**, **Django / DRF**,
**Flask**, **Ruby on Rails**, **Java / Spring Boot**.

Bilingual: the report is produced in **French** (default) or **English** — pass `fr`
or `en` to `/audit`.

## Install (marketplace)

```
/plugin marketplace add Dshellz/claude-code-security-audit
/plugin install security-audit
/audit
```

Then start a **new** Claude Code session and check with `/help` that `audit` appears.

## Usage

```
/audit                          # audit pending changes (git diff), auto-detect stack, FR report
/audit en                       # same, English report
/audit en src/Controller/Foo.php   # audit a specific file, English report
```

The report sorts findings by severity (`[CRITICAL]` / `[MAJOR]` / `[MINOR]` / `[INFO]`),
gives the risk, the vulnerable code and a fix for each, and ends with a one-line
conclusion (`All clear` or `N vulnerability(ies)`).

## How stack detection works

`/audit` looks for marker files at the project root: `composer.json` → PHP,
`package.json` with `next` → Next.js (else `express` → Node/Express), `manage.py` or a
`django` dependency → Django, a `flask` dependency → Flask, `Gemfile` → Rails,
`pom.xml` / `build.gradle` → Spring. If nothing matches, it asks.

## Repository layout

```
.claude-plugin/
└── marketplace.json              # marketplace catalog (one plugin)
plugins/
└── security-audit/               # the plugin
    ├── .claude-plugin/plugin.json
    ├── commands/audit.md         # /audit (auto-detection)
    └── rules/
        ├── report-template.fr.md
        ├── report-template.en.md
        ├── fr/  (php, node-express, nextjs, django, flask, rails, spring).md
        └── en/  (same 7 files)
```

Feel free to adapt the per-stack `rules/<lang>/<stack>.md` checklists to your business
logic and framework version.

## Limitations

This is a **review copilot**, not a static analyzer (SAST) or a pentest. An LLM can miss
a vulnerability or flag a false positive: keep a critical eye on the report and keep your
other safeguards (tests, SAST, human review). Only use this kit on code you are
responsible for.

## License

MIT.

---

🇫🇷 Version française : [README.fr.md](README.fr.md)
