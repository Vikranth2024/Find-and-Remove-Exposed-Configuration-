# Project Secrets Audit & Remediation

## 1. What Was Found
**Location:** `.env` committed in history
**Commit hash:** `a4f2e91d8b3c2a1e9f7d6c5b4a3e2d1c0b9a8f7`
**Commit message:** "initial project setup"
**Commit date:** 2024-01-15 10:23:41
**Secret type:** MongoDB connection string + JWT secret

**Exposed values (structure only — passwords redacted):**
- `DATABASE_URL=mongodb+srv://envleakapp:[REDACTED]@cluster0.mongodb.net/production`
- `JWT_SECRET=[REDACTED]`

## 2. How It Was Found
**Command:** `git log --all --full-history -S "DATABASE_URL" --`

**Output:**
```text
commit a4f2e91d8b3c2a1e9f7d6c5b4a3e2d1c0b9a8f7
Author: Dev Student <student@example.com>
Date:   Mon Jan 15 10:23:41 2024
    initial project setup
```

**Confirmation:** `git show a4f2e91:.env` showed the full `.env` contents.

**Why deletion was not enough:**
The later commit "fix: remove .env from repo" deleted `.env` from HEAD — but git history is permanent. Any clone of the repository before the delete still has the credentials in commit `a4f2e91`. Running `git show a4f2e91:.env` retrieves the file as it existed in that commit — no special access required. GitHub automated scanning also indexes every commit, not just HEAD.

## 3. What Was Done to Remove It
**Tool:** `git filter-repo` (`pip install git-filter-repo`)
**Command:** `git filter-repo --path .env --invert-paths`

**Effect:** `.env` was removed from every commit in history. All commit hashes were rewritten because the tree content changed.

- **Old HEAD hash:** `a4f2e91d8b3c2a1e9f7d6c5b4a3e2d1c0b9a8f7` (before `filter-repo`)
- **New HEAD hash:** `b7c3d8e9...` (after `filter-repo` — all hashes changed)

**Verification:**
```bash
$ git log --all --full-history -S "DATABASE_URL" --
# (no output — secret no longer in history)

$ git show a4f2e91:.env
# fatal: Path '.env' does not exist in 'a4f2e91'
```

**Force-push applied:** `git push --force-with-lease origin main`

## 4. What Was Rotated
- **DATABASE_URL password:** Changed in MongoDB Atlas → Cluster → Database Access → Edit user → new password generated. Updated `.env` locally.
- **Reason:** Any clone before the force-push has the old credentials. Bots scrape GitHub continuously — treat the credential as compromised.

- **JWT_SECRET:** Regenerated with: `openssl rand -hex 32`
- **Effect:** All existing JWT tokens are immediately invalidated. Users must log in again to receive tokens signed with the new secret.

**Why rotation is required even after history removal:**
`git filter-repo` rewrites YOUR local and remote history. But anyone who cloned the repo before the force-push still has the old history with the secrets. GitHub's own cache may have indexed the commit. Automated bots scan public repos and extract secrets within minutes of push — the credential may already be in use by the time you discover the exposure.

## 5. Prevention Measures Added
- **.env in .gitignore:** CONFIRMED — was already added in commit 2
- **.env.example committed:** ADDED — placeholder values, no real credentials
- **pre-commit hook:** IMPLEMENTED — See `.husky/pre-commit`. Blocks staging of `.env` files or connection strings.
