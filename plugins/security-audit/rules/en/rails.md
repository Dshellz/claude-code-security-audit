# Security audit rules - Ruby on Rails (white-box)

Inputs: `params`, headers, cookies.

## 1. Access control - IDOR (pitfall #1)

```ruby
# ❌ Invoice.find(params[:id])
# ✅ scoped to the user
current_user.invoices.find(params[:id])
```

Check authorization (Pundit/CanCanCan) at the object level, not just login.

## 2. Strong parameters / mass assignment

Always `params.require(:x).permit(:a, :b)`. Never `permit!` nor direct writing of
unfiltered `params` (otherwise `admin`, `role`… get written).

## 3. SQL injection

```ruby
# ❌ where("name = '#{params[:q]}'")
# ✅ parameterized
where("name = ?", params[:q])
```

Danger too with `find_by_sql`, `order(params[:sort])` (allow-list columns).

## 4. XSS

ERB auto-escapes; `html_safe` and `raw` on input = XSS. Enable a CSP.

## 5. CSRF

`protect_from_forgery` on by default → spot unjustified `skip_before_action` /
`skip_forgery_protection`.

## 6. RCE

Interpolation in `system`/`exec`/backticks, `eval`, `send(params[:m])` (arbitrary
method call), `Marshal.load`, `YAML.load` of input (→ `YAML.safe_load`).

## 7. Auth & sessions

`has_secure_password` (bcrypt). `ActiveSupport::SecurityUtils.secure_compare` for
tokens. Secrets via `credentials`/ENV (never committed).

## 8. Upload & path traversal

ActiveStorage with type/size validations; confine file paths.

## 9. Data exposure & logs

`filter_parameters` to avoid logging passwords/tokens. Don't expose sensitive fields in
views/JSON.

## 10. Open redirect & SSRF

`redirect_to params[:url]` → keep `allow_other_host: false` (recent default). Allow-list
for network requests to user URLs.

## 11. Config

`config.force_ssl`, `secret_key_base` from the environment, no debug mode in prod.

## 12. Dependencies

`bundler-audit`; `Gemfile.lock` committed.
