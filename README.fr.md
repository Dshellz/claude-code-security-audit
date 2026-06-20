# claude-code-security-audit

Un garde-fou de sécurité applicative (AppSec) pour **Claude Code**, prêt à l'emploi,
décliné par stack. Chaque bundle ajoute à ton projet une commande **`/audit`** qui fait
une revue de sécurité white-box de tes changements en cours (`git diff`) et rend un
rapport structuré, plus un mode « garde-fou passif » qui sécurise le code au fil de la
génération.

> Ce sont des **slash commands** Claude Code (`.claude/commands/audit.md`) plus un
> `CLAUDE.md` de projet, **pas** des « skills » Claude.

Disponible en deux langues : **`en/`** (anglais) et **`fr/`** (français). La langue des
fichiers du bundle détermine la langue du rapport d'audit.

## Stacks

| Stack                              | Chemin                                                  |
| ---------------------------------- | ------------------------------------------------------- |
| PHP (PDO/mysqli, Laravel, Symfony) | `<langue>/php/claude-code-audit-php/`                   |
| Node.js / Express                  | `<langue>/node-express/claude-code-audit-node-express/` |
| Next.js                            | `<langue>/nextjs/claude-code-audit-nextjs/`             |
| Django / DRF                       | `<langue>/django/claude-code-audit-django/`             |
| Flask                              | `<langue>/flask/claude-code-audit-flask/`               |
| Ruby on Rails                      | `<langue>/rails/claude-code-audit-rails/`               |
| Java / Spring Boot                 | `<langue>/spring/claude-code-audit-spring/`             |

## Installation

Copie le bundle de ta stack (le dossier `.claude/` **et** le fichier `CLAUDE.md`) à la
**racine de ton projet** - là où tu lances `claude` et où se trouve ton dépôt git.

```bash
# exemple pour un projet PHP, depuis la racine du projet
cp -r chemin/vers/fr/php/claude-code-audit-php/.claude .
cp    chemin/vers/fr/php/claude-code-audit-php/CLAUDE.md .
```

Si tu as déjà un `.claude/` ou un `CLAUDE.md`, **fusionne** au lieu d'écraser : pose
`audit.md` dans `.claude/commands/`, les deux autres `.md` dans `.claude/`, et recopie
le bloc « sécurité » dans ton `CLAUDE.md` existant.

Puis lance une **nouvelle session** Claude Code (les commandes sont détectées au
démarrage) et vérifie avec `/help` que `audit` apparaît.

## Utilisation

```
/audit                          # audite les changements en cours (git diff)
/audit src/Controller/Foo.php   # audite un fichier précis
```

Le rapport classe les findings par gravité (🔴 critique / 🟠 majeur / 🟡 mineur /
ℹ️ info), donne pour chacun le risque, le code vulnérable et le correctif, et finit
par `🟢 RAS` ou `🔴 N vulnérabilité(s)`.

## Contenu d'un bundle

```
claude-code-audit-<stack>/
├── CLAUDE.md                          # persona + garde-fou passif
└── .claude/
    ├── commands/audit.md              # la commande /audit
    ├── SECURITY_RULES.md              # checklist de vulnérabilités (par stack)
    └── security_report_template.md    # format du rapport
```

Adapte librement `SECURITY_RULES.md` à ta logique métier et à ta version de framework.

## Limites

C'est un **copilote de revue**, pas un scanner statique (SAST) ni un pentest. Un LLM
peut manquer une vulnérabilité ou en signaler une fausse : garde un œil critique sur le
rapport et conserve tes autres garde-fous (tests, SAST, revue humaine). N'utilise ce kit
que sur du code dont tu es responsable.

## Licence

MIT (suggestion - adapte selon ton choix).
