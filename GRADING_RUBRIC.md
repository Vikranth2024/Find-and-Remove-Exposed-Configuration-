# Evaluator Review Guide — Secrets Audit (Challenge 7.13)

This guide is for mentors and reviewers evaluating student submissions. It highlights what to look for and how to spot common "surface-level" solutions.

---

### 1. Artifact: SECRETS_AUDIT.md (Identification & Context)

**What Full Marks Look Like:**
Student provides the exact commit hash `a4f2e91d8b3c2a1e9f7d6c5b4a3e2d1c0b9a8f7` and explains that "deletion" only updates the HEAD of the branch, leaving the secret accessible to anyone who requests that specific historical object or commit.

```text
# Terminal evidence provided:
$ git show a4f2e91:.env
DATABASE_URL=mongodb+srv://envleakapp:P@ssw0rd123!@cluster0.mongodb.net/production
JWT_SECRET=super_secret_jwt_key_do_not_share_2024
```

**Common Partial-Credit Mistake:**
Identifying *when* it was deleted but failing to explain *why* that wasn't enough (i.e., simply saying "It was still in the history" without explaining how it's retrieved).

**The "Genuine Understanding" Check:**
Ask the student: *"If I have already cloned this repo locally before you performed the remediation, can I still see your secret after you force-push?"*
*Correct answer:* Yes, because the local clone still contains the old objects and history. Only a new clone or a prune/garbage collection would remove it locally.

---

### 2. Evidence: History Rewriting (git filter-repo)

**What Full Marks Look Like:**
Clear proof that the history was modified. This is evidenced by a change in HEAD commit hashes and an empty output from the grep/log commands.

```bash
# Terminal Evidence:
$ git log -S "DATABASE_URL"
# (Empty/No output)

$ git show a4f2e91:.env
fatal: Path '.env' does not exist in 'a4f2e91'
```

**Common Partial-Credit Mistake:**
Using `bfg-repo-cleaner` or `filter-branch` without correctly force-pushing, or failing to verify that *all* tags and branches were cleaned.

**The "Genuine Understanding" Check:**
Verify the old commit hash `a4f2e91` no longer contains the file. If the student only cleaned the current branch but left a `dev` or `staging` branch untouched, the secret still exists.

---

### 3. Prevention: Implementation of Controls

**What Full Marks Look Like:**
- `.env.example` contains all required keys but with generic placeholders.
- `.gitignore` specifically blocks `.env`.
- `.husky/pre-commit` script is functional and uses a regex to detect connection string patterns (e.g., `mongodb+srv://`).

**Common Partial-Credit Mistake:**
Committing a `.env.example` that still contains real (even if "rotated") credentials, or creating a Husky hook that only checks for the filename `.env` but doesn't check for staged secrets inside other files.

**The "Genuine Understanding" Check:**
Try to commit a file named `config.js` containing `mongodb+srv://user:pass@cluster.com`. A surface-level hook will pass it; a genuine security-minded hook will block the connection string pattern regardless of the filename.

---

### 4. Video/Live Demonstration (The Workflow)

**What Full Marks Look Like:**
A clear walkthrough showing:
1. `git log -S` finding the secret.
2. `git show` proving it's still accessible.
3. Execution of `git filter-repo`.
4. Verification that the secret is gone.
5. Verification that the Husky hook blocks a "test leak" attempt.

**Common Partial-Credit Mistake:**
Skipping the verification steps or failing to show the "before" state, making it impossible to confirm the tool actually worked.

**The "Genuine Understanding" Check:**
Look for the student running `git push --force-with-lease`. If they use `git push --force`, it's acceptable but `force-with-lease` demonstrates a higher level of DevOps maturity (avoiding overwriting others' work).
