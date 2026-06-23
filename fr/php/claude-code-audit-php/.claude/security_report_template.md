# Modèle de rapport d'audit

Le rapport produit par `/audit` doit suivre exactement cette structure.

````markdown
## Rapport d'audit de sécurité

**Périmètre :** <branche / fichiers / argument analysé>
**Synthèse :** X critique(s) · Y majeur(s) · Z mineur(s) · W info

---

### [CRITICAL] <Nom de la faille> — `chemin/du/fichier.php` (L<ligne>)

- **Où :** `chemin/du/fichier.php` (lignes X–Y)
- **Le risque :** Impact concret - qu'est-ce qu'un attaquant peut faire ?
- **Code vulnérable :**
  ```php
  <extrait minimal concerné>
  ```
- **Correction proposée :**
  ```php
  <code corrigé et sécurisé>
  ```
- **Pourquoi ça marche :** 1–2 phrases.

---

<un bloc par finding, du plus grave au moins grave>
````

## Échelle de gravité

Le badge est l'un de `[CRITICAL]` · `[MAJOR]` · `[MINOR]` · `[INFO]`, en majuscules.

- **[CRITICAL]** : exploitable à distance sans auth (ou avec un compte standard),
  menant à compromission / RCE / accès massif aux données d'autrui.
- **[MAJOR]** : faille réelle sous condition (auth requise, interaction, config).
- **[MINOR]** : durcissement / défense en profondeur.
- **[INFO]** : observation ou bonne pratique sans risque direct.

## Conclusion (obligatoire, une ligne)

- `RAS - aucune faille critique ou majeure détectée.`
- `N vulnérabilité(s) critique(s)/majeure(s) à corriger avant le merge.`
