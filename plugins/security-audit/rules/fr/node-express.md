# Règles d'audit de sécurité - Node.js / Express (white-box)

Entrées : `req.params`, `req.query`, `req.body`, `req.headers`, `req.cookies`.

## 1. Contrôle d'accès - IDOR / BOLA (piège n°1)

Express ne fournit rien par défaut : vérifier qu'un middleware d'authz s'applique à
CHAQUE route sensible, et l'appartenance des ressources.

```js
// ❌ const inv = await Invoice.findById(req.params.id);
// ✅ scope par l'utilisateur de session
const inv = await Invoice.findOne({ _id: req.params.id, ownerId: req.user.id });
if (!inv) return res.sendStatus(404);
```

Ne jamais accepter un champ `role`/`isAdmin` depuis le body.

## 2. Injection SQL / NoSQL

```js
// ❌ db.query(`SELECT * FROM users WHERE id = ${req.query.id}`)
// ✅ paramétré
db.query("SELECT * FROM users WHERE id = $1", [req.query.id]);
```

Knex : éviter `.raw()` concaténé. Mongoose : objet brut en filtre = injection
d'opérateurs (`$gt`, `$ne`) → valider/typer (`express-mongo-sanitize`).

## 3. XSS

Données utilisateur réfléchies dans du HTML sans échappement ; EJS `<%- %>` non
échappé. Laisser le moteur échapper ; assainir le HTML riche (DOMPurify) ; `helmet()` + CSP.

## 4. CSRF

Auth par cookie sans protection → `csurf` ou cookies `SameSite=Lax/Strict`.
Auth par en-tête `Authorization` (Bearer) = non auto-envoyée, moins exposée.

## 5. RCE / commandes

```js
// ❌ exec(`convert ${req.query.file} out.png`)
// ✅ pas de shell, arguments en tableau
execFile("convert", [file, "out.png"]);
```

Éviter `eval`, `new Function`, `vm`, `require()` d'un chemin utilisateur, et la
désérialisation non sûre (`node-serialize`).

## 6. Upload & path traversal

`multer` avec limites de taille/type, nom généré, chemin confiné (`path.resolve` sous
une racine), stockage hors dossier servi en statique.

## 7. Auth & sessions

`bcrypt`/`argon2` pour les mots de passe ; `crypto.timingSafeEqual` pour les secrets ;
JWT vérifié avec algorithme(s) fixé(s) (`algorithms:['HS256']`), expiration ; cookies
`httpOnly`, `secure`, `sameSite`.

## 8. Secrets & config

`process.env` + `.env` dans `.gitignore`. `cors()` sans options = toutes origines →
liste blanche. Gestionnaire d'erreurs qui ne fuit pas la stack en prod. `helmet()`.

## 9. Exposition & mass assignment

Ne pas renvoyer d'entités entières (hash, tokens). Éviter `Model.update(req.body)` /
`new Model(req.body)` → liste blanche de champs (`role`, `credits`…).

## 10. Validation

Valider toutes les entrées au bord (Zod, Joi, express-validator), refuser par défaut.

## 11. SSRF & redirection ouverte

Requête réseau vers une URL utilisateur (webhook, fetch) → liste blanche, bloquer IP
privées/métadonnées cloud. `res.redirect(req.query.url)` → chemins internes only.

## 12. Dépendances

`npm audit` ; lockfile présent.
