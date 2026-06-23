# Règles d'audit de sécurité - Next.js (App/Pages Router, white-box)

Le point critique en Next.js : la **frontière client / serveur**.

## 1. Frontière client / serveur & secrets

- Aucun secret dans un composant client ni dans une variable `NEXT_PUBLIC_*`
  (exposée au navigateur). Secrets côté serveur uniquement (`process.env.X`), idéalement
  avec le package `server-only` pour empêcher l'import côté client.
- Server Actions (`'use server'`) et Route Handlers reçoivent des arguments du client
  comme n'importe quelle API → **revalider** entrées **et** autorisation côté serveur.

## 2. Contrôle d'accès - IDOR (piège n°1)

Ne pas se fier au fait qu'un bouton est masqué dans l'UI. Dans chaque Server Action /
Route Handler, vérifier la session ET l'appartenance de la ressource.

```ts
// ✅
const session = await auth();
if (!session) return new Response("Unauthorized", { status: 401 });
const inv = await db.invoice.findFirst({
  where: { id, ownerId: session.user.id },
});
```

`middleware.ts` ne fait qu'un filtrage grossier : l'authz fine reste dans la couche données.

## 3. Injection SQL

Prisma/Drizzle paramètrent. Danger : `$queryRawUnsafe`, `$executeRawUnsafe`,
concaténation. Utiliser les requêtes paramétrées (`Prisma.sql`, tagged templates).

## 4. XSS

`dangerouslySetInnerHTML` avec données utilisateur → assainir (DOMPurify). En-têtes
de sécurité / CSP via `next.config.js` (`headers()`).

## 5. CSRF / redirection

Vérifier `SameSite` sur les cookies de session. `redirect(userInput)` → chemins
internes uniquement.

## 6. Upload, RCE, SSRF

Mêmes règles que Node : pas de `child_process` concaténé, valider les uploads, liste
blanche pour les fetch serveur vers des URL utilisateur (SSRF).

## 7. Auth & sessions

NextAuth/Auth.js : vérifier la session sur chaque route protégée (serveur). Cookies
`httpOnly`/`secure`/`sameSite`. JWT avec algorithme fixé.

## 8. Exposition & mass assignment

Ne pas renvoyer d'objets utilisateur complets vers le client (Server Components inclus).
Liste blanche des champs écrits depuis le client.

## 9. Validation

Zod sur les entrées des Server Actions et Route Handlers, refuser par défaut.

## 10. Dépendances

`npm audit` ; lockfile présent.
