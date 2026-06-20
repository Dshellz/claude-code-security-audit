# Règles d'audit de sécurité - Ruby on Rails (white-box)

Entrées : `params`, en-têtes, cookies.

## 1. Contrôle d'accès - IDOR (piège n°1)

```ruby
# ❌ Invoice.find(params[:id])
# ✅ portée par l'utilisateur
current_user.invoices.find(params[:id])
```

Vérifier les autorisations (Pundit/CanCanCan) au niveau objet, pas seulement le login.

## 2. Strong parameters / mass assignment

Toujours `params.require(:x).permit(:a, :b)`. Jamais de `permit!` ni d'écriture
directe de `params` non filtrés (sinon écriture de `admin`, `role`…).

## 3. Injection SQL

```ruby
# ❌ where("name = '#{params[:q]}'")
# ✅ paramétré
where("name = ?", params[:q])
```

Danger aussi sur `find_by_sql`, `order(params[:sort])` (whitelister les colonnes).

## 4. XSS

ERB auto-échappe ; `html_safe` et `raw` sur de l'entrée = XSS. Activer une CSP.

## 5. CSRF

`protect_from_forgery` actif par défaut → repérer les `skip_before_action` /
`skip_forgery_protection` injustifiés.

## 6. RCE

Interpolation dans `system`/`exec`/backticks, `eval`, `send(params[:m])` (appel de
méthode arbitraire), `Marshal.load`, `YAML.load` d'entrée (→ `YAML.safe_load`).

## 7. Auth & sessions

`has_secure_password` (bcrypt). `ActiveSupport::SecurityUtils.secure_compare` pour les
tokens. Secrets via `credentials`/ENV (jamais commités).

## 8. Upload & path traversal

ActiveStorage avec validations de type/taille ; confiner les chemins de fichiers.

## 9. Exposition & logs

`filter_parameters` pour ne pas logger mots de passe/tokens. Ne pas exposer de champs
sensibles dans les vues/JSON.

## 10. Redirection ouverte & SSRF

`redirect_to params[:url]` → garder `allow_other_host: false` (défaut récent). Liste
blanche pour les requêtes réseau vers des URL utilisateur.

## 11. Config

`config.force_ssl`, `secret_key_base` via l'environnement, pas de mode debug en prod.

## 12. Dépendances

`bundler-audit` ; `Gemfile.lock` versionné.
