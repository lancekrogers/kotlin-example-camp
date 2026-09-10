# Application security findings

These are defects in the code's own security properties. **None of them endanger the machine you
build on** — that question is answered in [findings-supply-chain.md](findings-supply-chain.md).
They matter because they would matter in any deployment, and because several sit directly in the
path of the exercise's feature work.

Context worth keeping in view: this is an abandoned 2019 sample app, and much of what follows is
placeholder-grade code that was never finished. That explains the defects; it does not make them
less real.

All line references are to `projects/kotlin-ktor-realworld-example-app`.

---

## B1 — Hardcoded JWT signing secret committed to the repository {#b1}

**Severity:** High · **Location:** `src/main/kotlin/io/realworld/app/utils/Cipher.kt:6`

```kotlin
val algorithm = Algorithm.HMAC256("something-very-secret-here")
```

This is the HMAC-SHA256 key that signs and verifies every authentication token, and it is public
in the repository — it is also the well-known Ktor sample placeholder, so it is in effect a
published constant.

Anyone holding it can mint a token that passes verification. The forged token needs
`issuer = "ktor-realworld"` (checked by `JwtProvider.verifier`, `JwtProvider.kt:14-17`),
`audience` containing `"ktor-audience"` (checked at `AppConfig.kt:76`), and an `email` claim
naming the victim (`AppConfig.kt:77`). All three values are constants in the source.

**Failure scenario:** an attacker signs `{issuer: "ktor-realworld", audience: ["ktor-audience"],
email: "victim@example.com"}` with the published key and presents it as
`Authorization: Token <jwt>`. `AppConfig.kt:77` resolves it to the victim's principal. Full
account takeover of any user, no credentials needed, no rate limit to defeat.

**Fix:** load the secret from configuration or environment, fail fast when it is absent, and never
commit a default.

---

## B2 — Passwords are stored as unsalted HMAC-SHA256 {#b2}

**Severity:** High · **Location:** `src/main/kotlin/io/realworld/app/domain/service/UserService.kt:19,25`
via `Cipher.kt:8-9`

```kotlin
userRepository.create(user.copy(password = String(base64Encoder.encode(Cipher.encrypt(user.password)))))
```

`Cipher.encrypt` is a single HMAC-SHA256, base64-encoded. That is a message authentication code,
not a password hashing function, and it fails on all three properties that matter:

- **No salt.** Identical passwords produce identical stored values across every account, so the
  database leaks which users share a password and is trivially attacked with a precomputed table.
- **No work factor.** One HMAC per guess means offline cracking runs at GPU speed — billions of
  candidates per second — against a stolen table.
- **The key is public** (B1), so the keyed construction contributes nothing. A pepper only helps
  while it is secret.

**Failure scenario:** the H2 database is disclosed. Every common password in it falls in seconds,
and the shared-password structure is visible without cracking anything.

**Fix:** bcrypt, scrypt, or Argon2id with per-user salts. This is a self-contained change
localized to `Cipher.kt` and `UserService.create`/`authenticate`.

---

## B3 — One key serves both password hashing and JWT signing {#b3}

**Severity:** High · **Location:** `Cipher.kt:6` used by both `JwtProvider.kt:15,19,28` and `UserService.kt:19,25`

`Cipher.algorithm` is the single `Algorithm` instance used to sign tokens *and* to derive stored
password values. Distinct cryptographic purposes should never share key material: it means one
disclosure compromises both systems at once, and it removes any possibility of rotating one
without breaking the other.

Here the key is already public, so the immediate incremental harm is small — but the structure is
wrong independently of B1, and fixing B1 alone (moving the secret to config) would leave a single
env var controlling both authentication and password storage.

**Fix:** separate key material per purpose. Adopting a real password KDF (B2) resolves this,
since the KDF carries its own per-user salt and no longer needs the JWT key.

---

## B4 — Password material is returned in API responses {#b4}

**Severity:** Medium-High · **Locations:** `UserService.kt:20,26,34`; `UserController.kt:15,23,36`; `User.kt:39-47`

The `User` data class carries `password` and has no `@JsonIgnore` on it, so Jackson serializes it
wherever a `User` is returned. Three endpoints do exactly that:

- **`POST /users` (register)** — `UserService.kt:20` returns `user.copy(token = …)` built from the
  *inbound* user, so the response echoes the **plaintext password** the client just submitted.
- **`POST /users/login`** — `UserService.kt:26` returns `userFound.copy(token = …)`, so the
  response carries the **stored HMAC value** for that account.
- **`GET /user` (current user)** — the principal is built by `UserService.getByEmail`
  (`UserService.kt:34`), which likewise retains the stored value; `UserController.kt:36` responds
  with it directly.

The RealWorld spec's User object is `{email, token, username, bio, image}` — password is not part
of it (`spec-api/README.md:27-37`).

**Failure scenario:** the stored value reaches anything that logs, caches, or proxies responses —
browser history, an APM trace, a CDN, a bug report. Because the values are unsalted (B2), a
disclosed one is directly attackable offline, and identical values across accounts are
immediately recognizable.

**Fix:** annotate `password` with `@JsonIgnore`, or introduce a response DTO that omits it. A
response DTO is the sturdier option — it stops the same leak recurring through `Article.author`,
which is also typed `User?` (`Article.kt:18`).

---

## B5 — `update()` writes the raw password to the database unhashed {#b5}

**Severity:** Medium · **Location:** `src/main/kotlin/io/realworld/app/domain/repository/UserRepository.kt:86-88`

```kotlin
if (user.password != null) {
    row[password] = user.password
}
```

`create()` passes the password through `Cipher.encrypt` before persisting (`UserService.kt:19`),
but the update path writes the value straight from the request body. Two consequences:

1. **Plaintext password at rest** for any user who updates their password — worse than the
   already-weak storage everywhere else.
2. **The account becomes unauthenticatable.** `authenticate()` (`UserService.kt:25`) compares the
   submitted password's HMAC against the stored value. After an update the stored value is
   plaintext, so the comparison can never match and the user is locked out.

The inconsistency between the two write paths is the root cause: hashing happens in the service
layer on create, and not at all on update.

**Fix:** hash in one place. Move it behind a single method both paths call, so the repository
never sees an unhashed password.

---

## B6 — Exception text is returned to clients and every error becomes a 500 {#b6}

**Severity:** Medium · **Location:** `src/main/kotlin/io/realworld/app/config/AppConfig.kt:82-89`

```kotlin
install(StatusPages) {
    exception(Exception::class.java) {
        val errorResponse = ErrorResponse(mapOf("error" to listOf("detail", this.toString())))
        context.respond(HttpStatusCode.InternalServerError, errorResponse)
    }
}
```

Two problems in four lines.

**Information disclosure.** `this.toString()` on the exception goes into the response body. For a
JDBC or Exposed failure that means SQL fragments, table and column names, and constraint details
reaching an unauthenticated caller — a map of the schema, free of charge.

**Every error is a 500.** The handler catches `Exception`, which is the supertype of the project's
own `UnauthorizedException` and `NotFoundException` (`domain/exceptions/`). A failed login
therefore returns `500` with `"email or password invalid!"` in the body instead of `401`, and a
missing user returns `500` instead of `404`. `ErrorExceptionMapping.kt` — which the name suggests
was meant to hold this mapping — is an empty object (`ErrorExceptionMapping.kt:5-6`).

This also breaks the spec, which requires 401/403/404 and a 422 validation shape
(`spec-api/README.md:166-187`), so it will fail the RealWorld spec tests the brief offers as a
bonus item.

**Failure scenario:** an attacker sends malformed input to any endpoint and reads schema details
out of the 500 body, then uses the 401-vs-500 collapse to distinguish nothing — which also means
legitimate clients cannot tell auth failures from server faults.

**Fix:** register per-exception handlers with correct status codes, and return a generic message
for unhandled exceptions while logging the detail server-side.

---

## B7 — An H2 Postgres-wire listener is opened on every startup {#b7}

**Severity:** Medium · **Location:** `src/main/kotlin/io/realworld/app/config/DbConfig.kt:10`

```kotlin
Server.createPgServer().start()
```

This starts H2's PostgreSQL-protocol server (default TCP **5435**) every time the app boots. The
application does not need it: Hikari connects through the JDBC URL on the line below
(`DbConfig.kt:11-16`), entirely in-process against an in-memory database.

So this is a listening network service with no purpose — pure added attack surface on the
developer's machine. H2's default is to refuse non-local connections, which keeps this at Medium
rather than higher, but the socket is still bound and the server is **never stopped** — there is
no shutdown hook and `DbConfig` exposes no teardown.

There is an operational edge too: `AppRule` (`src/test/.../rules/AppRule.kt:11`) calls `setup()`
per test class, so each test class start re-runs this line and attempts to bind 5435 again. That
is a likely source of port-conflict flakiness once the tests are re-enabled — which the exercise
requires.

**Fix:** delete the line. If a SQL console is genuinely wanted for local debugging, gate it behind
an explicit development flag and register a shutdown hook.

---

## B8 — Credential validation is inverted; only blank passwords pass {#b8}

**Severity:** High · **Location:** `src/main/kotlin/io/realworld/app/domain/User.kt:8-13,17-24,26-36`

```kotlin
fun validRegister(): User {
    require(
        user != null &&
            user.email.isEmailValid() &&
            user.password.isNullOrBlank() &&      // requires the password to BE blank
            user.username.isNullOrBlank()          // requires the username to BE blank
    ) { "User is invalid." }
    return user
}
```

The negation is missing. `require` throws unless its condition holds, so as written these
validators **demand that the password and username be empty**. All three are affected:
`validRegister()` (`:11-12`), `validLogin()` (`:21`), and `validToUpdate()` (`:30-33`, which
additionally requires `bio` and `image` to be blank).

The intent was plainly `!user.password.isNullOrBlank()`. Git history points at commit `be9efe7`,
"Changed the validation condition for register, login and update".

**Failure scenario:** a normal registration carrying a real password and username fails with
`"User is invalid."` — surfacing as a 500 via B6. Only a request with an empty password and empty
username is accepted, and `create()` then stores `Cipher.encrypt("")`, a fixed value shared by
every account so registered. Registration and login are both broken, and the only credentials the
system will accept are empty ones.

**Fix:** negate all six conditions. This is a one-line-per-check change, and it is a good
first-commit demonstration of the test gap — no test catches it, because every test class is
disabled (see the main audit's appendix, item 3).

---

## B9 — JWT hygiene {#b9}

**Severity:** Informational · **Location:** `src/main/kotlin/io/realworld/app/utils/JwtProvider.kt`

Three smaller observations, none independently actionable but worth recording:

- **10-hour validity with no revocation** (`:10`). Tokens carry an `email` claim and are accepted
  until expiry; there is no denylist, no session store, and no way to invalidate a token after a
  password change or a logout.
- **Inconsistent issuer checking** (`:14-17` vs `:19`). The `verifier` property applies
  `.withIssuer(issuer)`, but `decodeJWT` builds a fresh verifier without it. `decodeJWT` is
  currently unused, so nothing is exploitable today — but it is a loaded footgun for whoever calls
  it next.
- **Audience validated outside the verifier** (`AppConfig.kt:76`). The check lives in the Ktor
  `validate` block rather than in the JWT verifier itself, so any future code path that verifies
  tokens without going through Ktor authentication would skip it. `.withAudience()` on the
  verifier would make it structural.

These are hygiene items that only become material once B1 is fixed and the tokens are actually
worth forging.
