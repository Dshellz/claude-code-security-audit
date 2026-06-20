# Règles d'audit de sécurité - Java / Spring Boot (white-box)

Entrées : `@RequestParam`, `@PathVariable`, `@RequestBody`, en-têtes, cookies.

## 1. Contrôle d'accès - IDOR (piège n°1)

Vérifier l'appartenance, pas seulement l'authentification. `@PreAuthorize` /
sécurité de méthode, et filtrer par l'utilisateur courant.

```java
// ✅
var inv = invoiceRepo.findByIdAndOwner(id, currentUser);
```

Ne pas se reposer uniquement sur le matching d'URL de Spring Security.

## 2. Injection SQL

Spring Data / JPA paramètrent. Danger : requêtes natives concaténées, `@Query` avec
concaténation, `Statement` au lieu de `PreparedStatement`. Utiliser des paramètres liés.

## 3. XSS

Thymeleaf auto-échappe ; `th:utext` n'échappe pas. Encoder en sortie ; activer une CSP.

## 4. CSRF

Spring Security active CSRF pour les requêtes navigateur → repérer un `csrf().disable()`
injustifié sur des endpoints à cookie de session.

## 5. Désérialisation & RCE

Éviter la désérialisation Java native de données non fiables (`ObjectInputStream`).
Attention à l'injection SpEL, à `Runtime.exec`/`ProcessBuilder` concaténés.

## 6. Auth & sessions

`BCryptPasswordEncoder` (ou argon2). `MessageDigest.isEqual` pour comparaison à temps
constant. Cookies de session `HttpOnly`/`Secure`/`SameSite`.

## 7. Mass assignment

Ne pas binder l'entité directement (`@ModelAttribute` sur l'entité JPA) → utiliser des
DTO avec champs autorisés ; `@JsonIgnore` sur les champs sensibles.

## 8. Secrets & config

Secrets hors `application.properties` versionné → variables d'environnement / vault.
**Actuator** : ne pas exposer les endpoints de management sans protection. Pas de
stack trace détaillée en prod.

## 9. Upload, SSRF, redirection

Valider `MultipartFile` (type/taille), confiner les chemins. Liste blanche pour
`RestTemplate`/`WebClient` vers des URL utilisateur (SSRF). Open redirect sur
`sendRedirect(param)`.

## 10. Exposition & validation

Ne pas sérialiser les entités entières (hash, tokens). Bean Validation (`@Valid`,
`@NotNull`…) sur les entrées, refuser par défaut.

## 11. Dépendances

OWASP dependency-check / `mvn versions` ; mettre à jour les dépendances vulnérables.
