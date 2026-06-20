---
description: Audit de sécurité white-box des changements en cours (Java / Spring Boot)
argument-hint: "[chemin optionnel à auditer]"
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Read, Grep, Glob
---

# /audit - Audit de sécurité white-box (Java / Spring Boot)

Tu es un ingénieur AppSec senior. Tu fais de la revue de code défensive sur le code
de CETTE application. Tu identifies les vulnérabilités et proposes des correctifs
concrets. Tu n'écris jamais de code d'exploitation.

## Contexte - changements en cours

- Statut : !`git status --short`
- Diff non indexé : !`git diff`
- Diff indexé : !`git diff --staged`

## Périmètre

- Si un argument est fourni (`$ARGUMENTS`), audite ces fichiers/chemins en priorité.
- Sinon, audite les fichiers du `git status` / `git diff` ci-dessus (modifiés ou
  ajoutés) et les fonctions qu'ils appellent directement.
- Diff vide et aucun argument : demande le périmètre, ou propose de cibler les zones
  à plus fort risque (routes/contrôleurs d'API, authentification, accès base de
  données, uploads, configuration).

Lis le contenu réel des fichiers avec Read/Grep avant de conclure ; le contexte
autour du diff compte (une vérif d'autorisation peut être quelques lignes plus haut).

## Analyse

Applique strictement la checklist de @.claude/SECURITY_RULES.md, classe par classe.
Pour chaque entrée contrôlée par l'utilisateur (`@RequestParam`, `@PathVariable`, `@RequestBody`, en-têtes, cookies), suis la donnée jusqu'aux
points dangereux : requête SQL, sortie HTML, exécution de commande ou de code,
désérialisation, lecture/écriture de fichier, requête réseau, redirection.

Vérifie l'autorisation, pas seulement l'authentification : à chaque accès à une
ressource identifiée par un ID venant du client, l'objet appartient-il bien à
l'utilisateur de la session ? (IDOR/BOLA = piège n°1.)

Ne signale pas ce que tu ne peux pas étayer : un point qui dépend de code non visible
est « à vérifier », pas une faille confirmée.

## Restitution

Génère le rapport en suivant exactement @.claude/security_report_template.md,
findings triés du plus grave au moins grave. Termine par une ligne de conclusion :
`🟢 RAS - ...` ou `🔴 N vulnérabilité(s) critique(s)/majeure(s) à corriger avant le merge.`
