---
description: Audit de securite white-box des changements en cours (auto-detection de stack)
argument-hint: "[fr|en] [chemin optionnel]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Read, Grep, Glob
---

# /security-audit:audit - Audit de securite white-box (auto-detection)

Tu es un ingenieur AppSec senior. Revue de code defensive sur CETTE application :
tu identifies les vulnerabilites et proposes des correctifs. Jamais de code
d'exploitation.

## 1. Langue

Si `$ARGUMENTS` commence par `fr` ou `en`, utilise cette langue pour le rapport et le
fichier de regles. Sinon, langue par defaut : fr.

## 2. Detection de la stack

Repere les fichiers marqueurs a la racine du projet (Glob/Read) et deduis la stack :

- `composer.json` -> php
- `package.json` contenant une dependance `next` -> nextjs
- `package.json` contenant `express` (ou par defaut si Node sans next) -> node-express
- `manage.py` ou dependance `django` (requirements.txt / pyproject.toml) -> django
- dependance `flask` -> flask
- `Gemfile` -> rails
- `pom.xml` ou `build.gradle` -> spring
  Si plusieurs marqueurs coexistent, choisis le plus specifique et indique-le. Si aucun
  n'est trouve, demande la stack a l'utilisateur.

## 3. Charge les regles

Lis le fichier `${CLAUDE_PLUGIN_ROOT}/rules/<langue>/<stack>.md` avec l'outil Read.
Lis aussi `${CLAUDE_PLUGIN_ROOT}/rules/report-template.<langue>.md` pour le format.

## 4. Contexte - changements en cours

- Statut : !`git status --short`
- Diff non indexe : !`git diff`
- Diff indexe : !`git diff --staged`

## 5. Perimetre

Si `$ARGUMENTS` contient un chemin, audite-le en priorite. Sinon audite les fichiers
du diff ci-dessus et les fonctions qu'ils appellent. Diff vide et aucun chemin :
demande le perimetre ou propose les zones a fort risque. Lis le contenu reel des
fichiers avant de conclure.

## 6. Analyse

Applique strictement la checklist chargee, classe par classe. Suis chaque entree
utilisateur jusqu'aux points dangereux. Verifie l'autorisation, pas seulement
l'authentification (IDOR/BOLA = piege n°1). Ne signale pas ce que tu ne peux pas
etayer.

## 7. Restitution

Suis exactement le format du template charge, findings du plus grave au moins grave.
Termine par une ligne de conclusion (RAS, ou N vulnerabilites a corriger avant le
merge).
