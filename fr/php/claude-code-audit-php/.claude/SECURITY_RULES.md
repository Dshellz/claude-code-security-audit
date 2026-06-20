# Règles d'audit de sécurité - PHP (white-box)

Couvre PHP « brut » (PDO/mysqli), Laravel et Symfony. Adapte les exemples au
framework réellement utilisé dans le dépôt.

## 1. Contrôle d'accès - IDOR / BOLA (piège n°1)

Ressource chargée depuis un ID client sans vérifier l'appartenance.

```php
// ❌ $invoice = Invoice::find($request->id);
// ✅ Laravel
$invoice = $request->user()->invoices()->findOrFail($id);
$this->authorize('view', $invoice);
// ✅ PHP brut
$stmt = $pdo->prepare('SELECT * FROM invoices WHERE id = ? AND owner_id = ?');
$stmt->execute([$id, $_SESSION['user_id']]);
```

Rôles contrôlés serveur (pas un champ `role` lu du body), endpoints admin protégés.

## 2. Injection SQL

Concaténation dans une requête, `DB::raw`/`whereRaw` concaténés, DQL concaténé.

```php
// ❌ $pdo->query("SELECT * FROM users WHERE email = '$email'");
// ✅ requête préparée
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?'); $stmt->execute([$email]);
```

## 3. XSS

`echo` d'entrée sans `htmlspecialchars()`, Blade `{!! !!}`, Twig `|raw`.

```php
echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
```

HTML riche → HTML Purifier. Mettre une CSP.

## 4. CSRF

Mutations sans token. Laravel : routes retirant le middleware CSRF / `$except`. Cookies `SameSite`.

## 5. RCE PHP

`exec/shell_exec/system/passthru/popen/proc_open`, backticks, `eval`, `assert($input)`,
`preg_replace` modificateur `/e`.

- **Désérialisation** : `unserialize()` d'entrée = Object Injection (POP). Préférer
  `json_decode` ; sinon `['allowed_classes' => false]`.
- **LFI/RFI** : `include $_GET['page']` → liste blanche de fichiers.
  Éviter `extract($_REQUEST)` et `$$x` sur des entrées.

## 6. Upload & path traversal

Valider type réel (`finfo`), taille, renommer (`random_bytes`), stocker hors webroot,
confiner avec `realpath()`.

## 7. Auth & sessions

`password_hash`/`password_verify` (jamais md5/sha1). `hash_equals()` pour les tokens.
**Type juggling** : `===` et non `==` (magic hashes `0e...`). `session_regenerate_id(true)`
à la connexion ; cookies `httponly`, `secure`, `samesite`.

## 8. Secrets & config

Pas de secret en dur → `getenv()`/`.env` (`.gitignore`). `display_errors=Off`,
`APP_DEBUG=false`, `expose_php=Off`. CORS non permissif. En-têtes de sécurité.

## 9. Exposition & mass assignment

Ne pas sélectionner `*` (password, tokens) → `$hidden`/`select()`. `$fillable`
explicite ; éviter `Model::create($request->all())`. Pas de PII dans les logs.

## 10. Validation

Form Requests (Laravel) / Validator (Symfony) / `filter_var`. Refuser par défaut.

## 11. Redirection ouverte & injection d'en-têtes

`header('Location: '.$_GET['url'])`, injection CRLF, en-têtes `mail()` depuis entrées → liste blanche.

## 12. Dépendances

`composer audit` ; `composer.lock` versionné.
