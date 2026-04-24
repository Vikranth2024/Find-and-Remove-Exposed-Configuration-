# EnvLeakApp — Solution Reference

## Challenge: Secrets Audit (Deployment Challenge 7.13)

## What Was Found
File: `.env` committed in commit 1 ("initial project setup")
Secrets exposed:
- `DATABASE_URL`: MongoDB connection string with credentials
- `JWT_SECRET`: plaintext JWT signing key

## Full Remediation Steps

### Step 1 — Find
```bash
git log --all --full-history -S "DATABASE_URL" --
```
→ Returns: commit `a4f2e91...` "initial project setup"

```bash
git show a4f2e91:.env
```
→ Shows: `DATABASE_URL=mongodb+srv://envleakapp:P@ssw0rd123!@...`

### Step 2 — Remove
```bash
pip install git-filter-repo
git filter-repo --path .env --invert-paths
```

### Step 3 — Verify
```bash
git log --all --full-history -S "DATABASE_URL" --
```
→ Returns: (nothing)

```bash
git show a4f2e91:.env
```
→ `fatal: Path '.env' does not exist in 'a4f2e91'`

### Step 4 — Force-Push
```bash
git remote add origin <fork-url>
git push --force-with-lease origin main
```

### Step 5 — Prevent
- `.env.example` created
- `.gitignore` already contains `.env` (was added in commit 2)

## Files in This Solution
- `SECRETS_AUDIT.md` (complete, all five sections filled)
- `.env.example` (placeholder values)
- `.gitignore` (verified correct)
- `.husky/pre-commit` (optional bonus)
