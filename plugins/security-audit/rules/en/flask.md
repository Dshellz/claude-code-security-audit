# Security audit rules - Flask (white-box)

Flask is minimal: CSRF, headers and sessions must be wired up by hand.

## 1. SSTI (Flask/Jinja2-specific pitfall)

`render_template_string(user_input)` or a template built with input = SSTI → RCE.
Always render fixed templates and pass data as a **variable**.

```python
# ❌ render_template_string("Hello " + request.args['name'])
# ✅ render_template("hello.html", name=request.args['name'])
```

## 2. Access control - IDOR (pitfall #1)

Check the session and resource ownership (`owner_id == session['user_id']`). Don't expose
a sensitive endpoint without an auth decorator.

## 3. SQL injection

SQLAlchemy parameterizes; danger with concatenated `text()` and raw cursors. Use bound
queries / the ORM.

## 4. Debug in prod = RCE

`app.run(debug=True)` exposes the Werkzeug console (code execution). Never in prod.

## 5. RCE

`subprocess(..., shell=True)`, `os.system`, `eval`, `pickle.loads`, `yaml.load` of input.

## 6. CSRF

No native protection → Flask-WTF (`CSRFProtect`) on mutations. `SameSite` cookies.

## 7. Upload & path traversal

`werkzeug.utils.secure_filename`, validate type/size, store outside the static folder,
confine paths.

## 8. Auth & sessions

`generate_password_hash`/`check_password_hash`; `hmac.compare_digest` for tokens.
Strong `SECRET_KEY` from the environment (otherwise session cookies are forgeable).
Cookies `httponly`/`secure`/`samesite`.

## 9. Secrets & config

Secrets out of the repo. Security headers (Flask-Talisman or manual). No stack trace
exposed in prod.

## 10. Data exposure & validation

Don't return sensitive fields. Validate inputs (`pydantic`/schemas), deny by default.

## 11. SSRF & open redirect

`requests.get(user_url)` → allow-list. Open redirect on `redirect(request.args[...])`.

## 12. Dependencies

`pip-audit` / `safety`; pinned versions.
