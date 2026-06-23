# claude-code-security-audit

![Démo de la commande /security-audit:audit](docs/demo-plugin.gif)

Un plugin de sécurité applicative (AppSec) prêt à l'emploi pour **Claude Code**. Il
ajoute une commande **`/security-audit:audit`** qui **auto-détecte ta stack**, applique la checklist de
vulnérabilités correspondante, fait une revue de sécurité white-box de tes changements en
cours (`git diff`) et rend un rapport structuré.

Stacks supportées : **PHP**, **Node.js / Express**, **Next.js**, **Django / DRF**,
**Flask**, **Ruby on Rails**, **Java / Spring Boot**.

Bilingue : le rapport est produit en **français** (par défaut) ou en **anglais** -
passe `fr` ou `en` à `/security-audit:audit`.

## Utilisation

```
/security-audit:audit                          # audite les changements en cours (git diff), auto-détection, rapport FR
/security-audit:audit en                       # idem, rapport en anglais
/security-audit:audit fr src/Controller/Foo.php   # audite un fichier précis, rapport FR
```

Le rapport classe les findings par gravité (`[CRITICAL]` / `[MAJOR]` / `[MINOR]` /
`[INFO]`), donne pour chacun le risque, le code vulnérable et le correctif, et finit par
une ligne de conclusion (`RAS` ou `N vulnérabilité(s)`).

## Comment marche la détection de stack

`/security-audit:audit` repère les fichiers marqueurs à la racine du projet : `composer.json` → PHP,
`package.json` avec `next` → Next.js (sinon `express` → Node/Express), `manage.py` ou une
dépendance `django` → Django, une dépendance `flask` → Flask, `Gemfile` → Rails,
`pom.xml` / `build.gradle` → Spring. Si rien ne correspond, il demande.

## Arborescence du dépôt

```
.claude-plugin/
└── marketplace.json              # catalogue marketplace (un plugin)
plugins/
└── security-audit/               # le plugin
    ├── .claude-plugin/plugin.json
    ├── commands/audit.md         # /security-audit:audit (auto-détection)
    └── rules/
        ├── report-template.fr.md
        ├── report-template.en.md
        ├── fr/  (php, node-express, nextjs, django, flask, rails, spring).md
        └── en/  (mêmes 7 fichiers)
```

Adapte librement les checklists par stack `rules/<langue>/<stack>.md` à ta logique métier
et à ta version de framework.

## Installation (marketplace)

```
/plugin marketplace add Dshellz/claude-code-security-audit
/plugin install security-audit
/security-audit:audit
```

Puis lance une **nouvelle session** Claude Code et vérifie avec `/help` que `audit`
apparaît.

> À noter : ce dépôt est désormais **100% plugin**. Les anciens bundles par stack que tu
> copiais dans `.claude/` ont été supprimés ; installe via la marketplace à la place.

## Limites

C'est un **copilote de revue**, pas un scanner statique (SAST) ni un pentest. Un LLM
peut manquer une vulnérabilité ou en signaler une fausse : garde un œil critique sur le
rapport et conserve tes autres garde-fous (tests, SAST, revue humaine). N'utilise ce kit
que sur du code dont tu es responsable.

## Licence

MIT.
