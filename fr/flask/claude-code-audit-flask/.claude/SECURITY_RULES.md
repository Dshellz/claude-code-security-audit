# Règles d'audit de sécurité - Flask (white-box)

Flask est minimaliste : CSRF, en-têtes, sessions sont à câbler à la main.

## 1. SSTI (piège spécifique Flask/Jinja2)

`render_template_string(user_input)` ou un template construit avec de l'entrée = SSTI
→ RCE. Toujours rendre des templates fixes et passer la donnée en **variable**.

```python
# ❌ render_template_string("Hello " + request.args['name'])
# ✅ render_template("hello.html", name=request.args['name'])
```

## 2. Contrôle d'accès - IDOR (piège n°1)

Vérifier la session et l'appartenance de la ressource (`owner_id == session['user_id']`).
Ne pas exposer d'endpoint sensible sans décorateur d'auth.

## 3. Injection SQL

SQLAlchemy paramètre ; danger sur `text()` concaténé et les curseurs bruts. Utiliser
les requêtes liées / l'ORM.

## 4. Debug en prod = RCE

`app.run(debug=True)` expose la console Werkzeug (exécution de code). Jamais en prod.

## 5. RCE

`subprocess(..., shell=True)`, `os.system`, `eval`, `pickle.loads`, `yaml.load` d'entrée.

## 6. CSRF

Pas de protection native → Flask-WTF (`CSRFProtect`) sur les mutations. Cookies `SameSite`.

## 7. Upload & path traversal

`werkzeug.utils.secure_filename`, valider type/taille, stocker hors dossier statique,
confiner les chemins.

## 8. Auth & sessions

`generate_password_hash`/`check_password_hash` ; `hmac.compare_digest` pour les tokens.
`SECRET_KEY` fort et via l'environnement (sinon cookies de session forgeables).
Cookies `httponly`/`secure`/`samesite`.

## 9. Secrets & config

Secrets hors dépôt. En-têtes de sécurité (Flask-Talisman ou manuels). Pas de stack
trace exposée en prod.

## 10. Exposition & validation

Ne pas renvoyer de champs sensibles. Valider les entrées (`pydantic`/schémas),
refuser par défaut.

## 11. SSRF & redirection ouverte

`requests.get(user_url)` → liste blanche. Open redirect sur `redirect(request.args[...])`.

## 12. Dépendances

`pip-audit` / `safety` ; versions épinglées.
