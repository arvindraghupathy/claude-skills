# Example Flow

A complete walkthrough of `/step-by-step` from activation to first test. Reference this to
see what good behaviour looks like at each phase.

```
User: /step-by-step Add authentication to the API endpoints

Claude: Step-by-step mode on. Before I touch anything, I need to understand
this properly. A few questions:

1. Which endpoints need to be protected — all of them, or specific ones?
2. What auth mechanism are you using — JWT, sessions, API keys, OAuth?
3. Is there an existing auth library/service in the codebase, or from scratch?

---

User: All endpoints except /health. JWT. There's already a jwt library installed
but no middleware yet.

Claude: Got it. One more thing — what should happen when auth fails? 401 with
a JSON error body? And should the token come from the Authorization header,
a cookie, or either?

---

User: 401 with JSON. Authorization header only.

Claude: Here's my understanding of what we're doing:

**Goal:** Protect all API endpoints except /health with JWT auth via Authorization
         header. Failed auth returns 401 JSON. No middleware exists yet.
**Scope:** Middleware layer + route registration. No changes to business logic.
**Approach:** Create auth middleware following existing patterns, apply to all
         routes except /health.
**Risks:** /health must stay public; existing routes shouldn't break.

Does this match what you have in mind?

💬 Good time to run `/compact` before we plan.

---

User: Yes exactly.

Claude: [reads codebase]

Here's the plan:

1. **Create auth middleware** — Add /middleware/auth.js with a validateToken function
   following the pattern in /middleware/logging.js
2. **Register middleware on protected routes** — Apply to all routes in /routes/api.js
   except /health
3. **Add error response helper** — Add formatAuthError to /utils/errors.js so the 401
   matches the existing error shape

Does this breakdown look right? I won't start until you approve it.

---

User: Looks good.

Claude: **Step 1 of 3:** Create auth middleware
No middleware exists yet. I'll create /middleware/auth.js with a validateToken
function that reads the Authorization header and verifies the JWT, following
the same module.exports pattern as /middleware/logging.js.

Making the change now...

✅ Done. Created /middleware/auth.js. Extracts the Bearer token, verifies it with
the existing jwt library, attaches the decoded payload to req.user, calls next().
On failure, calls next(err) with a structured error — ready for the helper in step 3.

🔍 Run a code review on this step?
  Y) Yes   N) No — continue

---

[Steps 2 and 3 complete]

Claude: All code changes are done. Before we test — do any of these need attention
for the changes we just made?
  • Error handling  • Logging  • Analytics/telemetry  • Performance
Let me know which (if any) matter here, or "none".

---

User: Add logging on auth failures.

Claude: [adds one atomic logging step, executes, reports]

Now, test scenarios. Here's what I'd cover:

1. **Valid token — request passes** — valid JWT populates req.user, calls next()
2. **Missing Authorization header** — 401 with the standard error shape
3. **Malformed header** — "Token abc" instead of "Bearer abc" → 401
4. **Expired token** — valid-but-expired JWT → 401, not 500
5. **Tampered token** — invalid signature → 401
6. **/health stays public** — no token → passes through

Want to add, remove, or change any of these?

Also — how would you like to review the tests?
  A) Per test — pause after each   B) All at end — present together
```
