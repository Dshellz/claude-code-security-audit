# Règles d'audit de sécurité - Django / DRF (white-box)

Entrées : `request.GET`, `request.POST`, `request.data` (DRF), kwargs d'URL.

## 1. Contrôle d'accès - IDOR (piège n°1)

```python
# ❌ Invoice.objects.get(pk=pk)
# ✅ portée par l'utilisateur
Invoice.objects.get(pk=pk, owner=request.user)
```

DRF : définir `permission_classes` ; ne pas exposer un queryset global. Vérifier les
permissions d'objet, pas seulement `IsAuthenticated`.

## 2. Injection SQL

ORM paramétré. Danger : `.raw()`, `.extra()`, `RawSQL` concaténés, `cursor.execute`
avec `%` formaté. Toujours passer les paramètres séparément.

## 3. XSS

Templates auto-échappés ; `|safe`, `mark_safe`, `format_html` mal utilisés désactivent
la protection. Activer une CSP.

## 4. CSRF

Middleware CSRF actif par défaut → repérer `@csrf_exempt` injustifié et les vues API
non protégées qui reposent sur des cookies de session.

## 5. RCE

`os.system`, `subprocess(..., shell=True)`, `eval`, `pickle.loads` d'entrée,
`yaml.load` (→ `yaml.safe_load`). Éviter l'exécution dynamique.

## 6. Upload & path traversal

Valider type/taille, ne pas faire confiance au nom fourni, stocker via `storage`,
empêcher `../` dans les chemins.

## 7. Auth & sessions

`make_password`/`check_password` (hasher fort) ; `constant_time_compare` pour les
tokens ; `SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`, `SECURE_HSTS_*`,
`SESSION_COOKIE_HTTPONLY`.

## 8. Secrets & config (prod)

`DEBUG = False`, `SECRET_KEY` via l'environnement, `ALLOWED_HOSTS` restreint,
secrets hors dépôt. Pas de page d'erreur détaillée exposée.

## 9. Exposition & mass assignment

`ModelForm`/serializers : déclarer `fields` explicitement, **jamais** `'__all__'` sur
des modèles à champs sensibles. Ne pas sérialiser le hash de mot de passe.

## 10. Validation

Formes/serializers pour valider ; refuser par défaut.

## 11. SSRF & redirection ouverte

`requests.get(user_url)` → liste blanche, bloquer IP internes. `redirect(request.GET['next'])`
→ `url_has_allowed_host_and_scheme`.

## 12. Dépendances

`pip-audit` / `safety` ; versions épinglées.
