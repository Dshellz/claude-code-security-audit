# Security audit rules - Node.js / Express (white-box)

Inputs: `req.params`, `req.query`, `req.body`, `req.headers`, `req.cookies`.

## 1. Access control - IDOR / BOLA (pitfall #1)

Express provides nothing by default: make sure an authz middleware applies to EVERY
sensitive route, and check resource ownership.

```js
// ❌ const inv = await Invoice.findById(req.params.id);
// ✅ scope by session user
const inv = await Invoice.findOne({ _id: req.params.id, ownerId: req.user.id });
if (!inv) return res.sendStatus(404);
```

Never accept a `role`/`isAdmin` field from the body.

## 2. SQL / NoSQL injection

```js
// ❌ db.query(`SELECT * FROM users WHERE id = ${req.query.id}`)
// ✅ parameterized
db.query("SELECT * FROM users WHERE id = $1", [req.query.id]);
```

Knex: avoid concatenated `.raw()`. Mongoose: a raw object as a filter = operator
injection (`$gt`, `$ne`) → validate/type (`express-mongo-sanitize`).

## 3. XSS

User data reflected into HTML without escaping; EJS `<%- %>` is unescaped. Let the
engine escape; sanitize rich HTML (DOMPurify); `helmet()` + CSP.

## 4. CSRF

Cookie-based auth without protection → `csurf` or `SameSite=Lax/Strict` cookies.
Header-based auth (`Authorization` Bearer) is not auto-sent, less exposed.

## 5. RCE / commands

```js
// ❌ exec(`convert ${req.query.file} out.png`)
// ✅ no shell, arguments as an array
execFile("convert", [file, "out.png"]);
```

Avoid `eval`, `new Function`, `vm`, `require()` of a user path, and unsafe
deserialization (`node-serialize`).

## 6. Upload & path traversal

`multer` with size/type limits, generated name, confined path (`path.resolve` under a
root), storage outside any statically served folder.

## 7. Auth & sessions

`bcrypt`/`argon2` for passwords; `crypto.timingSafeEqual` for secrets; JWT verified
with fixed algorithm(s) (`algorithms:['HS256']`), expiration; cookies `httpOnly`,
`secure`, `sameSite`.

## 8. Secrets & config

`process.env` + `.env` in `.gitignore`. `cors()` with no options = all origins →
allow-list. Error handler that doesn't leak the stack in prod. `helmet()`.

## 9. Data exposure & mass assignment

Don't return whole entities (hash, tokens). Avoid `Model.update(req.body)` /
`new Model(req.body)` → field allow-list (`role`, `credits`…).

## 10. Validation

Validate all inputs at the edge (Zod, Joi, express-validator), deny by default.

## 11. SSRF & open redirect

Network request to a user URL (webhook, fetch) → allow-list, block private IPs / cloud
metadata. `res.redirect(req.query.url)` → internal paths only.

## 12. Dependencies

`npm audit`; lockfile present.
