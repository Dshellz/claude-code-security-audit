# Security audit rules - Django / DRF (white-box)

Inputs: `request.GET`, `request.POST`, `request.data` (DRF), URL kwargs.

## 1. Access control - IDOR (pitfall #1)

```python
# ❌ Invoice.objects.get(pk=pk)
# ✅ scoped to the user
Invoice.objects.get(pk=pk, owner=request.user)
```

DRF: set `permission_classes`; don't expose a global queryset. Check object-level
permissions, not just `IsAuthenticated`.

## 2. SQL injection

ORM is parameterized. Danger: concatenated `.raw()`, `.extra()`, `RawSQL`,
`cursor.execute` with `%` formatting. Always pass parameters separately.

## 3. XSS

Templates auto-escape; misused `|safe`, `mark_safe`, `format_html` disable the
protection. Enable a CSP.

## 4. CSRF

CSRF middleware on by default → spot unjustified `@csrf_exempt` and unprotected API
views relying on session cookies.

## 5. RCE

`os.system`, `subprocess(..., shell=True)`, `eval`, `pickle.loads` of input,
`yaml.load` (→ `yaml.safe_load`). Avoid dynamic execution.

## 6. Upload & path traversal

Validate type/size, don't trust the supplied name, store via `storage`, prevent `../`
in paths.

## 7. Auth & sessions

`make_password`/`check_password` (strong hasher); `constant_time_compare` for tokens;
`SESSION_COOKIE_SECURE`, `CSRF_COOKIE_SECURE`, `SECURE_HSTS_*`, `SESSION_COOKIE_HTTPONLY`.

## 8. Secrets & config (prod)

`DEBUG = False`, `SECRET_KEY` from the environment, restricted `ALLOWED_HOSTS`, secrets
out of the repo. No detailed error page exposed.

## 9. Data exposure & mass assignment

`ModelForm`/serializers: declare `fields` explicitly, **never** `'__all__'` on models
with sensitive fields. Don't serialize the password hash.

## 10. Validation

Forms/serializers to validate; deny by default.

## 11. SSRF & open redirect

`requests.get(user_url)` → allow-list, block internal IPs. `redirect(request.GET['next'])`
→ `url_has_allowed_host_and_scheme`.

## 12. Dependencies

`pip-audit` / `safety`; pinned versions.
