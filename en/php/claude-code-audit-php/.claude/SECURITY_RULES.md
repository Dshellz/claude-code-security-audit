# Security audit rules - PHP (white-box)

Covers plain PHP (PDO/mysqli), Laravel and Symfony. Adapt the examples to the
framework actually used in the repo.

## 1. Access control - IDOR / BOLA (pitfall #1)

Resource loaded from a client-supplied ID without an ownership check.

```php
// ❌ $invoice = Invoice::find($request->id);
// ✅ Laravel
$invoice = $request->user()->invoices()->findOrFail($id);
$this->authorize('view', $invoice);
// ✅ plain PHP
$stmt = $pdo->prepare('SELECT * FROM invoices WHERE id = ? AND owner_id = ?');
$stmt->execute([$id, $_SESSION['user_id']]);
```

Roles enforced server-side (not a `role` field read from the body), admin endpoints protected.

## 2. SQL injection

String concatenation in a query, concatenated `DB::raw`/`whereRaw`, concatenated DQL.

```php
// ❌ $pdo->query("SELECT * FROM users WHERE email = '$email'");
// ✅ prepared statement
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = ?'); $stmt->execute([$email]);
```

## 3. XSS

`echo` of input without `htmlspecialchars()`, Blade `{!! !!}`, Twig `|raw`.

```php
echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');
```

Rich HTML → HTML Purifier. Add a CSP.

## 4. CSRF

Mutations without a token. Laravel: routes removing the CSRF middleware / `$except`. `SameSite` cookies.

## 5. PHP RCE

`exec/shell_exec/system/passthru/popen/proc_open`, backticks, `eval`, `assert($input)`,
`preg_replace` `/e` modifier.

- **Deserialization**: `unserialize()` of input = Object Injection (POP chains). Prefer
  `json_decode`; otherwise `['allowed_classes' => false]`.
- **LFI/RFI**: `include $_GET['page']` → file allow-list.
  Avoid `extract($_REQUEST)` and `$$x` on inputs.

## 6. Upload & path traversal

Validate real type (`finfo`), size, rename (`random_bytes`), store outside the webroot,
confine with `realpath()`.

## 7. Auth & sessions

`password_hash`/`password_verify` (never md5/sha1). `hash_equals()` for tokens.
**Type juggling**: use `===` not `==` (magic hashes `0e...`). `session_regenerate_id(true)`
on login; cookies `httponly`, `secure`, `samesite`.

## 8. Secrets & config

No hardcoded secret → `getenv()`/`.env` (`.gitignore`). `display_errors=Off`,
`APP_DEBUG=false`, `expose_php=Off`. Non-permissive CORS. Security headers.

## 9. Data exposure & mass assignment

Don't `SELECT *` (password, tokens) → `$hidden`/`select()`. Explicit `$fillable`;
avoid `Model::create($request->all())`. No PII in logs.

## 10. Validation

Form Requests (Laravel) / Validator (Symfony) / `filter_var`. Deny by default.

## 11. Open redirect & header injection

`header('Location: '.$_GET['url'])`, CRLF injection, `mail()` headers from input → allow-list.

## 12. Dependencies

`composer audit`; `composer.lock` committed.
