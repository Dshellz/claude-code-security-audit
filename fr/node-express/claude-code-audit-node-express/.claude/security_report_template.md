# Modèle de rapport d'audit

Le rapport produit par `/audit` doit suivre exactement cette structure.

````markdown
## 🚨 Rapport d'audit de sécurité

**Périmètre :** <branche / fichiers / argument analysé>
**Synthèse :** 🔴 X critique(s) · 🟠 Y majeur(s) · 🟡 Z mineur(s) · ℹ️ W info

---

### [Gravité] - <Nom de la faille> - `chemin/du/fichier.js`

- **Où :** `chemin/du/fichier.js` (lignes X–Y)
- **Le risque :** Impact concret - qu'est-ce qu'un attaquant peut faire ?
- **Code vulnérable :**
  ```js
  <extrait minimal concerné>
  ```
- **Correction proposée :**
  ```js
  <code corrigé et sécurisé>
  ```
- **Pourquoi ça marche :** 1–2 phrases.

---

<un bloc par finding, du plus grave au moins grave>
````

## Échelle de gravité

- **🔴 Critique** : exploitable à distance sans auth (ou avec un compte standard),
  menant à compromission / RCE / accès massif aux données d'autrui.
- **🟠 Majeur** : faille réelle sous condition (auth requise, interaction, config).
- **🟡 Mineur** : durcissement / défense en profondeur.
- **ℹ️ Info** : observation ou bonne pratique sans risque direct.

## Conclusion (obligatoire, une ligne)

- `🟢 RAS - aucune faille critique ou majeure détectée.`
- `🔴 N vulnérabilité(s) critique(s)/majeure(s) à corriger avant le merge.`
