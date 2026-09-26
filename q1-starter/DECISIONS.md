# DECISIONS

### Production static files use a Windows-safe filesystem path

**What I chose:**

Use `fileURLToPath(new URL('../dist/', import.meta.url))` for the production `dist` directory in `server/index.js`.

**Why:**

The production Playwright server initially returned `NOT_FOUND` for `/`, even though `dist/index.html` existed. Checking the generated URL pathname showed it began with `/D:/...` on Windows. After changing the path conversion to `fileURLToPath(...)`, the targeted Playwright test `the shell carries the active org identity` passed.

**What I rejected:**

Using `new URL('../dist/', import.meta.url).pathname` directly. On Windows this produced a pathname beginning with `/D:/`, which was not the filesystem path expected by the path utilities used by `serveStatic()`.

**What would change my mind:**

Evidence from another supported runtime where `fileURLToPath(...)` caused an incorrect filesystem path or broke production static-file serving.


### Database seed loading uses a URL instead of `.pathname`

**What I chose:**

Use the `URL` object returned by `new URL(...)` directly when resolving the database seed path in `scripts/load-db.js`.

**Why:**

The original code used `.pathname`, which produced an invalid Windows path such as `D:\D:\...` when the script was executed. `node scripts/load-db.js` failed before the API suite could run. After changing the helper to return the `URL` directly, the database seeded successfully and `node scripts/check-api.js` passed all 66 tests.

**What I rejected:**

Keeping `.pathname` and handling the resulting Windows path manually. The failure showed that the URL pathname was not suitable as a cross-platform filesystem path.

**What would change my mind:**

A reproducible failure showing that the URL form is incompatible with another supported runtime or causes the seed script to resolve the wrong database file.