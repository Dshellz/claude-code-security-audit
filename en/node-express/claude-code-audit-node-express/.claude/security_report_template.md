# Audit report template

The report produced by `/audit` must follow this exact structure.

````markdown
## 🚨 Security audit report

**Scope:** <branch / files / argument analyzed>
**Summary:** 🔴 X critical · 🟠 Y major · 🟡 Z minor · ℹ️ W info

---

### [Severity] - <Vulnerability name> - `path/to/file.js`

- **Where:** `path/to/file.js` (lines X–Y)
- **Risk:** Concrete impact - what can an attacker do?
- **Vulnerable code:**
  ```js
  <minimal relevant snippet>
  ```
- **Proposed fix:**
  ```js
  <fixed, secure code>
  ```
- **Why it works:** 1–2 sentences.

---

<one block per finding, most to least severe>
````

## Severity scale

- **🔴 Critical**: remotely exploitable without auth (or with a standard account),
  leading to compromise / RCE / mass access to other users' data.
- **🟠 Major**: real flaw under a condition (auth required, interaction, config).
- **🟡 Minor**: hardening / defense in depth.
- **ℹ️ Info**: observation or best practice with no direct risk.

## Conclusion (mandatory, one line)

- `🟢 All clear - no critical or major finding detected.`
- `🔴 N critical/major vulnerability(ies) to fix before merge.`
