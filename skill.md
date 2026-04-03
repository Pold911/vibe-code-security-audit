---
name: vibe-code-security-audit
description: >
  Audit web applications and codebases for the most common and dangerous security
  vulnerabilities — especially those introduced by AI-assisted ("vibe coded") development.
  Use this skill whenever the user asks to review code for security issues, harden an app,
  audit an API, check for vulnerabilities, or secure a project. Also trigger when the user
  mentions terms like "security review", "pentest checklist", "harden my app",
  "is my code secure", "fix security holes", "OWASP", "SQL injection", "XSS",
  "vibe code security", or shares backend/frontend code and asks if anything looks wrong.
  Even if the user just says "review my code" without mentioning security, consider
  triggering this skill — security is always relevant.
---

# Vibe-Code Security Audit

Systematic security audit for web applications, with special attention to vulnerabilities
that AI code-generation tools introduce most frequently.

Source: [@hartdrawss](https://x.com/hartdrawss/status/2039998901176897860)

## Audit workflow

### Step 1 — Gather context

Before diving in, understand the stack:
- Framework and language (Express, Django, Rails, Next.js, etc.)
- Monolith or separate frontend/backend?
- Deployment target (Vercel, AWS, self-hosted, etc.)
- Authentication method (JWT, sessions, OAuth)
- Database type

Infer what you can from the code itself. Don't interrogate the user if the answers are obvious.

### Step 2 — Scan for the top 20 vulnerabilities

Work through each item in the checklist below against the user's code or architecture.
These are ordered by how often they appear in AI-generated code, not by severity.

For each vulnerability found:
- Explain **what's wrong** in plain language (assume the developer may be junior).
- Show the **offending code** if identifiable.
- Provide a **concrete fix** with a code example.
- Rate severity: 🔴 Critical, 🟠 High, 🟡 Medium, 🔵 Low.

If no code is provided, walk the user through the checklist interactively and ask
targeted questions to identify likely issues.

### Step 3 — Report findings

Lead with the most dangerous issues. End with a summary count by severity and a
prioritized action list. Only report actual findings — skip items that look clean.

### Step 4 — Suggest structural improvements

Look for patterns suggesting systemic issues:
- No middleware pattern → suggest a security middleware layer
- No input validation → suggest a validation library (zod, joi, etc.)
- No env management → suggest dotenv + .gitignore patterns
- No dependency auditing → suggest `npm audit` or `pip-audit` in CI

## Severity definitions

- 🔴 **Critical** — Exploitable now, no authentication required. Data breach or full system compromise likely. Examples: SQL injection, exposed DB ports, hardcoded frontend API keys, server running as root.
- 🟠 **High** — Exploitable with some effort or preconditions. Significant data exposure or privilege escalation possible. Examples: no rate limiting on auth, IDOR, JWTs in localStorage, weak JWT secrets.
- 🟡 **Medium** — Requires specific conditions or chaining with another vulnerability. Examples: CORS wildcard, missing HTTPS, verbose errors, open redirects.
- 🔵 **Low** — Best practice violation that increases attack surface but isn't directly exploitable alone. Examples: unaudited npm packages, sessions not invalidated on logout, non-expiring tokens.

## Tone

Be direct. If something is broken, say so clearly. But be constructive — every problem
gets a solution. Avoid jargon walls; explain *why* something is dangerous, not just
*that* it is. Many developers doing AI-assisted coding are newer to security concepts.

---

# The 20-item checklist

## 1. API keys hardcoded in frontend JavaScript

Keys embedded in client-side code are visible to anyone who opens devtools. AI coding
tools do this constantly because they optimize for "it works" not "it's secure."

An exposed key gives attackers access to whatever service it controls.

Move all keys to the backend. The frontend should never hold secrets.

```javascript
// ❌ Key visible to anyone
const res = await fetch('https://api.example.com/data', {
  headers: { 'Authorization': `Bearer ${API_KEY}` }
});

// ✅ Frontend calls your backend, backend holds the key
const res = await fetch('/api/data');
```

## 2. No rate limiting on authentication endpoints

The `/login` endpoint accepts unlimited requests. Bots can try thousands of
username/password combinations unimpeded.

Add rate limiting and account lockout after 5 failed attempts. This is table stakes.

```javascript
const rateLimit = require('express-rate-limit');
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts. Try again later.'
});
app.post('/login', loginLimiter, loginHandler);
```

## 3. SQL queries built with string concatenation

`"SELECT * FROM users WHERE id=" + userId` is textbook SQL injection. An attacker can
read, modify, or delete any data in the database.

Use parameterized queries. Every database library supports them.

```javascript
// ❌ SQL injection
const query = "SELECT * FROM users WHERE id=" + userId;

// ✅ Parameterized
const query = "SELECT * FROM users WHERE id = $1";
const result = await db.query(query, [userId]);
```

## 4. CORS set to wildcard (*)

`Access-Control-Allow-Origin: *` means any website can make requests to your API.
Combined with credentials, any malicious site can make authenticated requests using
your users' cookies.

Whitelist specific origins only.

```javascript
// ❌ Any site can call your API
app.use(cors({ origin: '*' }));

// ✅ Explicit whitelist
app.use(cors({
  origin: ['https://myapp.com', 'https://staging.myapp.com'],
  credentials: true
}));
```

## 5. JWTs stored in localStorage

localStorage is readable by any JavaScript on the page. One XSS vulnerability
anywhere on your site lets an attacker steal every user's token.

Use httpOnly cookies instead — they're inaccessible to JavaScript.

```javascript
res.cookie('token', jwt, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 3600000
});
```

## 6. Weak or default JWT secrets

If the signing secret is `"secret"`, `"password"`, or copied from a tutorial,
attackers will guess it. They test common secrets first. Your secret is probably
on a wordlist already.

Generate a cryptographically random 256-bit secret. Rotate it periodically.

```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

Store it in an environment variable, never in source code.

## 7. Admin routes protected only in the frontend

React Router guards are cosmetic. The server doesn't care about them. Hit the API
endpoint directly with curl and it opens right up.

Protect every route server-side. Frontend guards are for UX only.

```javascript
function requireAdmin(req, res, next) {
  if (!req.user || req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' });
  }
  next();
}
app.get('/api/admin/users', requireAdmin, getUsers);
```

## 8. .env file committed to git history

Even if the file was later deleted, it's in the git history forever.

```bash
git log --all --full-history -- .env
```

If this returns results, rotate every key that was ever in that file immediately.
Add `.env` to `.gitignore`. Consider `git-secrets` or `trufflehog` to scan history.

## 9. Error responses exposing internals

Stack traces, database table names, file paths, and framework versions in error
responses give attackers a map of your infrastructure.

Log full errors server-side. Return generic messages to the client.

```javascript
// ❌ Handing attackers a blueprint
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message, stack: err.stack });
});

// ✅ Keep internals internal
app.use((err, req, res, next) => {
  console.error(err);
  res.status(500).json({ error: 'Internal server error' });
});
```

## 10. File uploads with no MIME type validation

Extension checks alone don't protect you. An attacker can upload a server-side script
disguised with an innocent extension and gain full server access.

Validate MIME type server-side using the file's magic bytes, not the filename.
Store uploads outside the web root.

```javascript
const fileType = require('file-type');
async function validateUpload(buffer) {
  const type = await fileType.fromBuffer(buffer);
  const allowed = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
  if (!type || !allowed.includes(type.mime)) {
    throw new Error('Invalid file type');
  }
}
```

## 11. Passwords hashed with MD5 or SHA1

Rainbow tables crack MD5 in seconds. No salt means no protection.

Use bcrypt or Argon2. No exceptions.

```javascript
const bcrypt = require('bcrypt');
const hash = await bcrypt.hash(password, 12);
const match = await bcrypt.compare(password, hash);
```

## 12. Authentication tokens that never expire

A stolen token with no expiry grants permanent access forever.

Set short expiry on access tokens. Implement refresh token rotation.

```javascript
const token = jwt.sign({ userId: user.id }, secret, { expiresIn: '15m' });
const refreshToken = jwt.sign({ userId: user.id }, refreshSecret, { expiresIn: '7d' });
```

Issue a new refresh token each time one is used and invalidate the old one.

## 13. Auth middleware missing on internal API routes

AI tools add middleware to obvious routes and skip the rest. One unprotected
endpoint is all it takes.

Apply middleware globally and opt-out for public routes, not the other way around.

```javascript
// ✅ Protect everything by default
app.use('/api', authMiddleware);

// Explicitly mark public routes
app.get('/api/public/health', skipAuth, healthCheck);
```

Audit every single endpoint manually. Assume nothing is protected until verified.

## 14. Application server running as root

One exploit equals full system access. Run the app as a non-privileged user.
This costs nothing to fix.

```bash
useradd -r -s /bin/false appuser
su -s /bin/bash -c 'node server.js' appuser
```

In Docker, add `USER appuser` to the Dockerfile.

## 15. Database port exposed to the internet

Your PostgreSQL on port 5432 should never have a public IP. Attackers scan for
open database ports constantly.

Put it behind a firewall or private network. This is a one-click fix in most
cloud providers.

## 16. IDOR vulnerabilities on resource endpoints

Change the ID in the URL. Can you access another user's data? In most vibe-coded
apps: yes. Code generators focus on CRUD, not authorization.

Validate ownership server-side on every resource request.

```javascript
app.get('/api/orders/:id', auth, async (req, res) => {
  const order = await Order.findById(req.params.id);
  if (!order || order.userId !== req.user.id) {
    return res.status(404).json({ error: 'Not found' });
  }
  res.json(order);
});
```

Return 404 instead of 403 to avoid confirming the resource exists.

## 17. No HTTPS enforcement

Credentials sent over plain HTTP can be intercepted on any public network.

Enforce HTTPS at the server level. Redirect all HTTP traffic.

```javascript
app.use((req, res, next) => {
  if (req.headers['x-forwarded-proto'] !== 'https') {
    return res.redirect(301, `https://${req.headers.host}${req.url}`);
  }
  next();
});
```

Set HSTS headers to prevent downgrade attacks.

## 18. Sessions not invalidated on logout

Clearing the cookie client-side is not enough. The old session token still works
on the server after the user clicks logout.

Invalidate sessions server-side on every logout event.

```javascript
app.post('/logout', auth, async (req, res) => {
  await Session.destroy({ where: { token: req.token } });
  res.clearCookie('token');
  res.json({ message: 'Logged out' });
});
```

## 19. npm packages not audited since setup

Run `npm audit` right now. Count the criticals. Schedule this as part of every deploy.

```bash
npm audit
npm audit fix

# Python equivalent
pip-audit
```

## 20. Open redirects in callback URLs

Attackers use open redirects to send users to phishing sites through your trusted
domain. The URL looks legitimate because it starts with your domain.

Validate and whitelist every redirect destination. Never trust user-supplied
redirect URLs.

```javascript
const ALLOWED_REDIRECTS = ['/dashboard', '/profile', '/settings'];
app.get('/callback', (req, res) => {
  const redirect = req.query.redirect || '/dashboard';
  if (!ALLOWED_REDIRECTS.includes(redirect)) {
    return res.redirect('/dashboard');
  }
  res.redirect(redirect);
});
```
