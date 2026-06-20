# Security audit rules - Java / Spring Boot (white-box)

Inputs: `@RequestParam`, `@PathVariable`, `@RequestBody`, headers, cookies.

## 1. Access control - IDOR (pitfall #1)

Check ownership, not just authentication. `@PreAuthorize` / method security, and filter
by the current user.

```java
// ✅
var inv = invoiceRepo.findByIdAndOwner(id, currentUser);
```

Don't rely solely on Spring Security's URL matching.

## 2. SQL injection

Spring Data / JPA parameterize. Danger: concatenated native queries, `@Query` with
concatenation, `Statement` instead of `PreparedStatement`. Use bound parameters.

## 3. XSS

Thymeleaf auto-escapes; `th:utext` does not. Encode on output; enable a CSP.

## 4. CSRF

Spring Security enables CSRF for browser requests → spot an unjustified `csrf().disable()`
on session-cookie endpoints.

## 5. Deserialization & RCE

Avoid native Java deserialization of untrusted data (`ObjectInputStream`). Beware SpEL
injection and concatenated `Runtime.exec`/`ProcessBuilder`.

## 6. Auth & sessions

`BCryptPasswordEncoder` (or argon2). `MessageDigest.isEqual` for constant-time
comparison. Session cookies `HttpOnly`/`Secure`/`SameSite`.

## 7. Mass assignment

Don't bind the entity directly (`@ModelAttribute` on the JPA entity) → use DTOs with
allowed fields; `@JsonIgnore` on sensitive fields.

## 8. Secrets & config

Secrets out of a committed `application.properties` → environment variables / vault.
**Actuator**: don't expose management endpoints without protection. No detailed stack
trace in prod.

## 9. Upload, SSRF, redirect

Validate `MultipartFile` (type/size), confine paths. Allow-list for `RestTemplate`/
`WebClient` to user URLs (SSRF). Open redirect on `sendRedirect(param)`.

## 10. Data exposure & validation

Don't serialize whole entities (hash, tokens). Bean Validation (`@Valid`, `@NotNull`…)
on inputs, deny by default.

## 11. Dependencies

OWASP dependency-check / `mvn versions`; update vulnerable dependencies.
