# Pre-Publication Verification Checklist

Before pushing your MCP Server to public GitHub, use this checklist to verify everything is properly configured for secure public release.

## Files Created / Verified ✓

- [ ] `.gitignore` exists in root directory
  - [ ] Contains `dbhub.toml` (prevents committing actual config)
  - [ ] Contains `dbhub/.env` (prevents committing env vars)
  - [ ] Contains `dbhub/certs/*.pem` (prevents committing SSL keys)
  
- [ ] `dbhub.toml.example` exists in root directory
  - [ ] Uses placeholders like `[YOUR_DATABASE_HOST]`
  - [ ] Has comments explaining each configuration option
  - [ ] Includes SSH tunnel example (commented out)

- [ ] `dbhub/.env.example` exists in dbhub directory
  - [ ] Shows format for DSN method
  - [ ] Shows format for individual parameters (with special char example)
  - [ ] Has optional configuration examples

- [ ] `SETUP.md` exists in root directory
  - [ ] Contains quick start (5 steps)
  - [ ] Explains TOML configuration method
  - [ ] Explains .env configuration method
  - [ ] Contains SSH tunnel instructions
  - [ ] Contains security best practices
  - [ ] Contains troubleshooting section

- [ ] `dbhub/README.md` updated
  - [ ] References SETUP.md for first-time setup

---

## Sensitive Data Check ✓

### Check your git status:

```bash
git status
```

Verify that these files are NOT listed (should be gitignored):
- [ ] `dbhub.toml` - should NOT appear
- [ ] `dbhub/.env` - should NOT appear
- [ ] `dbhub/certs/cert.pem` - should NOT appear
- [ ] `dbhub/certs/key.pem` - should NOT appear

### Check git history (one-time only):

If you've previously committed these files, clean them up:

```bash
# Check if sensitive files are in git history
git log --all --full-history -- dbhub.toml
git log --all --full-history -- dbhub/.env
```

If found, remove them:
```bash
# Remove dbhub.toml from history
git filter-branch --tree-filter 'rm -f dbhub.toml' -- --all

# Or use BFG Repo-Cleaner if filter-branch is too slow
bfg --delete-files dbhub.toml
bfg --delete-files '*.pem'

# Force push (use with caution - notifies all collaborators)
git push origin --force --all
```

---

## Content Verification ✓

### Example Files Have Placeholders:

Check `dbhub.toml.example`:
```bash
grep -E "\[YOUR_|<your?_|actual_" dbhub.toml.example
```
Should show placeholder entries like:
- `[YOUR_DATABASE_HOST]`
- `[YOUR_DATABASE_PASSWORD]`
- `[YOUR_DATABASE_USERNAME]`

Check that actual values are NOT present:
```bash
# These should return NOTHING if you properly replaced with placeholders
grep "172\.16\." dbhub.toml.example
grep "GPL-STL" dbhub.toml.example
grep "akm123" dbhub.toml.example
```

### Documentation is Complete:

- [ ] SETUP.md is 150+ lines with comprehensive instructions
- [ ] .gitignore covers all sensitive file types
- [ ] Example files have clear comments
- [ ] No credentials in any example files

---

## Simulated User Experience ✓

Test the experience of a new user cloning your repository:

```bash
# In a temporary directory
cd /tmp
git clone <your-repo-url> test-clone
cd test-clone

# Verify structure
ls -la | grep -E "(SETUP|\.gitignore|dbhub\.toml)"
# Should show:
# - SETUP.md ✓
# - .gitignore ✓
# - dbhub.toml.example ✓
# - dbhub.toml (should NOT exist) ✓

# Check that actual config files don't exist
test -f dbhub.toml && echo "ERROR: dbhub.toml exists!" || echo "✓ dbhub.toml properly gitignored"
test -f dbhub/.env && echo "ERROR: .env exists!" || echo "✓ .env properly gitignored"
test -f dbhub/certs/key.pem && echo "ERROR: key.pem exists!" || echo "✓ certs properly gitignored"

# Verify user can read setup instructions
head -20 SETUP.md  # Should show helpful introduction
```

---

## Repository README Update ✓

Consider adding this section to your main repository README:

```markdown
## Getting Started

For setup and configuration instructions, see [SETUP.md](./SETUP.md).

### Quick Start

1. Clone this repository
2. Copy `dbhub.toml.example` to `dbhub.toml`
3. Edit `dbhub.toml` with your database credentials
4. Follow [SETUP.md](./SETUP.md) for detailed instructions

### Security

- Never commit `dbhub.toml` or `dbhub/.env` to version control
- These files are protected by `.gitignore`
- See [SETUP.md](./SETUP.md) for security best practices
```

---

## CI/CD Pipeline Check ✓

If you use GitHub Actions or other CI/CD:

- [ ] Secrets are stored in CI/CD secrets manager, not in code
- [ ] `.gitignore` prevents committing `.env` files
- [ ] No test data contains real credentials
- [ ] Database integration tests use test databases, not production

---

## Final Checks Before Pushing ✓

```bash
# 1. Verify .gitignore is working
git check-ignore dbhub.toml
git check-ignore dbhub/.env  
git check-ignore dbhub/certs/key.pem
# All three should return the path (meaning they're ignored)

# 2. Verify no sensitive data in last commit
git show HEAD:dbhub.toml.example | grep -i "password\|credential\|secret" | grep -v "^\[" | grep -v "^#"
# Should return nothing or only comments

# 3. Check all important docs exist
test -f SETUP.md && echo "✓ SETUP.md found" || echo "✗ SETUP.md missing"
test -f .gitignore && echo "✓ .gitignore found" || echo "✗ .gitignore missing"
test -f dbhub.toml.example && echo "✓ dbhub.toml.example found" || echo "✗ dbhub.toml.example missing"
test -f dbhub/.env.example && echo "✓ .env.example found" || echo "✗ .env.example missing"

# 4. Verify git will not track sensitive files
git status
# dbhub.toml, dbhub/.env, dbhub/certs/*.pem should NOT appear

# 5. Count lines (sanity check example files are non-trivial)
wc -l SETUP.md dbhub.toml.example dbhub/.env.example
```

---

## Ready to Publish! 

When all checkboxes are complete, you're ready to:

```bash
git add .
git commit -m "Prepare public release: Add configuration examples and setup guide"
git push origin main
```

Your repository is now safe for public distribution! Users can:
1. ✓ Clone the repository without seeing credentials
2. ✓ Follow SETUP.md to configure their own instance
3. ✓ Use example files as templates
4. ✓ Never accidentally commit sensitive data

---

## Emergency: Credentials Accidentally Committed?

If you discover you accidentally committed credentials to a public repository:

### Immediate Actions:
1. **Rotate all credentials** - Change database passwords, disable API keys, etc.
2. **Clean git history** - Use BFG Repo-Cleaner or git filter-branch
3. **Force push** - Update remote repository
4. **Notify users** - If others cloned, let them know to pull latest

### Tools:
- [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) - Fast and safe
- `git filter-branch` - Built-in but slower
- [git-secrets](https://github.com/awslabs/git-secrets) - Prevent future commits

