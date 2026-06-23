# Security audit rules - Next.js (App/Pages Router, white-box)

The critical point in Next.js: the **client / server boundary**.

## 1. Client / server boundary & secrets

- No secret in a client component or a `NEXT_PUBLIC_*` variable (exposed to the
  browser). Server-side secrets only (`process.env.X`), ideally with the `server-only`
  package to prevent client-side import.
- Server Actions (`'use server'`) and Route Handlers receive client arguments like any
  API → **re-validate** inputs **and** authorization on the server.

## 2. Access control - IDOR (pitfall #1)

Don't rely on a button being hidden in the UI. In each Server Action / Route Handler,
check the session AND resource ownership.

```ts
// ✅
const session = await auth();
if (!session) return new Response("Unauthorized", { status: 401 });
const inv = await db.invoice.findFirst({
  where: { id, ownerId: session.user.id },
});
```

`middleware.ts` only does coarse filtering: fine-grained authz stays in the data layer.

## 3. SQL injection

Prisma/Drizzle parameterize. Danger: `$queryRawUnsafe`, `$executeRawUnsafe`,
concatenation. Use parameterized queries (`Prisma.sql`, tagged templates).

## 4. XSS

`dangerouslySetInnerHTML` with user data → sanitize (DOMPurify). Security headers / CSP
via `next.config.js` (`headers()`).

## 5. CSRF / redirect

Check `SameSite` on session cookies. `redirect(userInput)` → internal paths only.

## 6. Upload, RCE, SSRF

Same rules as Node: no concatenated `child_process`, validate uploads, allow-list for
server fetches to user URLs (SSRF).

## 7. Auth & sessions

NextAuth/Auth.js: verify the session on every protected route (server). Cookies
`httpOnly`/`secure`/`sameSite`. JWT with fixed algorithm.

## 8. Data exposure & mass assignment

Don't return whole user objects to the client (including in Server Components). Allow-list
client-writable fields.

## 9. Validation

Zod on Server Action and Route Handler inputs, deny by default.

## 10. Dependencies

`npm audit`; lockfile present.
