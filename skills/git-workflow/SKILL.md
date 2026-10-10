---
name: git-workflow
description: Use when working with Git, creating commits, branches, or pull requests. Enforces Conventional Commits, Git Flow branching, PR guidelines, and code review standards.
version: 1.0.0
updated: 2026-06-21
---

# Git Workflow & Standards

## ⚠️ PERINGATAN — HARUS DIIKUTI

**Aturan di skill ini WAJIB dipatuhi.** Branching strategy dan commit message standards bukan sekadar saran, melainkan keharusan untuk menjaga integritas code history dan kelancaran kolaborasi/deployment.

🔴 **Konsekuensi jika dilanggar:**
- Commit message asal-asalan → history tidak terbaca, tracking perubahan sulit
- Branching tidak disiplin → konflik kode saat merge, regression bug di production
- Commit file sensitif → credentials bocor ke public repository (resiko keamanan tinggi)
- Force push sembarangan → hilangnya progress kerja developer lain

Model **WAJIB** membaca dan mematuhi setiap aturan di bawah.

---

## Tujuan
Memastikan version control yang terorganisir, readable git history, dan collaboration yang efektif dalam tim.

## Branching Strategy

### Git Flow (untuk production apps)

```
main (production)
  ├── develop (integration)
  │   ├── feature/user-authentication
  │   ├── feature/payment-gateway
  │   └── feature/notification-system
  ├── release/v1.2.0
  ├── hotfix/critical-security-patch
```

**Branch Types:**

1. **main** (atau **master**)
   - Production-ready code
   - Every commit adalah release
   - Protected branch (no direct commits)

2. **develop**
   - Integration branch
   - Latest development changes
   - Source untuk feature branches

3. **feature/***
   - New features
   - Branch dari develop
   - Merge back ke develop
   - Naming: `feature/nama-fitur`

4. **release/***
   - Preparation untuk production release
   - Bug fixes only, no new features
   - Naming: `release/v1.2.0`

5. **hotfix/***
   - Critical fixes untuk production
   - Branch dari main
   - Merge ke main DAN develop
   - Naming: `hotfix/deskripsi-singkat`

### Workflow Commands

```bash
# Start new feature
git checkout develop
git pull origin develop
git checkout -b feature/user-authentication

# Work on feature
git add .
git commit -m "feat: implement user login endpoint"

# Keep feature branch updated
git checkout develop
git pull origin develop
git checkout feature/user-authentication
git rebase develop

# Finish feature
git checkout develop
git merge --no-ff feature/user-authentication
git push origin develop
git branch -d feature/user-authentication
```

## Commit Message Standards

### Format (Conventional Commits)

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type

- **feat**: Fitur baru
- **fix**: Bug fix
- **docs**: Perubahan dokumentasi
- **style**: Formatting, semicolons, dll (no code change)
- **refactor**: Code refactoring (no bug fix, no feature)
- **perf**: Performance improvement
- **test**: Menambah atau memperbaiki tests
- **chore**: Maintenance tasks, dependency updates
- **ci**: CI/CD configuration changes
- **build**: Build system changes

### Examples

```bash
# Good commits
feat(auth): menambahkan endpoint login dengan JWT
fix(payment): memperbaiki perhitungan diskon yang salah
docs(api): memperbarui dokumentasi endpoint users
refactor(database): mengoptimalkan query products
perf(api): menambahkan caching untuk dashboard stats
test(auth): menambahkan unit test untuk login service

# With body
feat(notification): menambahkan sistem notifikasi email

Implementasi:
- Email service dengan queue
- Template email untuk berbagai event
- Retry mechanism untuk failed emails

Closes #123

# Breaking change
feat(api)!: mengubah struktur response API

BREAKING CHANGE: Response structure berubah dari { data, status } 
menjadi { success, data, message }
```

### Subject Rules

- Gunakan bahasa Indonesia BAKU
- Maksimal 72 karakter
- Tidak diakhiri dengan titik
- Gunakan imperative mood: "menambahkan" bukan "menambah" atau "ditambahkan"
- Deskriptif dan jelas

## Pull Request (PR) / Merge Request (MR)

### PR Title

Format sama dengan commit message:
```
feat(auth): menambahkan fitur two-factor authentication
```

### PR Description Template

```markdown
## Deskripsi

Jelaskan perubahan yang dibuat dan mengapa.

## Tipe Perubahan

- [ ] Bug fix (non-breaking change)
- [ ] Fitur baru (non-breaking change)
- [ ] Breaking change (fix atau feature yang menyebabkan existing functionality tidak berfungsi)
- [ ] Dokumentasi

## Testing

Jelaskan testing yang sudah dilakukan:

- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing di local environment
- [ ] Tested di staging environment

## Checklist

- [ ] Code mengikuti style guidelines
- [ ] Self-review sudah dilakukan
- [ ] Dokumentasi sudah diperbarui
- [ ] Tidak ada warning atau error baru
- [ ] Tests ditambahkan (jika applicable)
- [ ] Dependent changes sudah di-merge

## Screenshots (jika applicable)

Sebelum:
[screenshot]

Sesudah:
[screenshot]

## Related Issues

Closes #123
Fixes #456
Related to #789
```

### PR Best Practices

1. **Small & Focused**
   - Satu PR = satu feature/fix
   - < 400 lines changed (ideal)
   - Easy to review

2. **Clear Description**
   - Jelaskan "what" dan "why"
   - Include screenshots/videos untuk UI changes
   - Link related issues

3. **Self Review**
   - Review own PR sebelum request review
   - Check diff, no debug code, no commented code

4. **Tests Included**
   - Add tests untuk new features
   - Update tests untuk changes

## Code Review Guidelines

### Sebagai Reviewer

1. **Review Promptly**
   - Review dalam 24 jam
   - Jangan block progress

2. **Be Constructive**
   - Explain "why" dalam feedback
   - Suggest alternatives
   - Praise good code

3. **Focus On**
   - Logic errors
   - Security issues
   - Performance concerns
   - Code maintainability
   - Test coverage

4. **Comments Types**
   ```
   ❗ CRITICAL: Must fix (security, bugs)
   ⚠️  CONCERN: Should fix (performance, best practices)
   💡 SUGGESTION: Nice to have
   ❓ QUESTION: Need clarification
   ✅ APPROVAL: Looks good
   ```

### Sebagai Author

1. **Respond to All Comments**
   - Address concerns
   - Explain decisions
   - Mark as resolved

2. **Don't Take It Personally**
   - Code review ≠ personal criticism
   - Learn from feedback

3. **Update PR**
   - Fix issues
   - Add tests
   - Update documentation

## Git Best Practices

### .gitignore

```gitignore
# Environment
.env
.env.local
.env.production

# Dependencies
/node_modules
/vendor

# Build outputs
/dist
/build
/public/build

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Logs
*.log
/storage/logs

# Secrets
*.key
*.pem
secrets/
```

### Git Hooks

```bash
# .git/hooks/pre-commit
#!/bin/bash

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "❌ Linting failed. Please fix errors."
    exit 1
fi

# Run tests
npm run test
if [ $? -ne 0 ]; then
    echo "❌ Tests failed. Please fix failing tests."
    exit 1
fi

echo "✅ Pre-commit checks passed"
```

### Useful Git Commands

```bash
# Amend last commit (hanya untuk unpushed commits)
git commit --amend -m "New commit message"

# Interactive rebase (clean up commits)
git rebase -i HEAD~3

# Stash changes
git stash
git stash pop

# Cherry pick commit
git cherry-pick <commit-hash>

# Reset to specific commit (DANGEROUS!)
git reset --hard <commit-hash>

# Revert commit (safe)
git revert <commit-hash>

# View file history
git log --follow -p -- path/to/file

# Find who changed line
git blame path/to/file

# Search in git history
git log -S "search term" --source --all
```

## Tagging & Releases

```bash
# Semantic Versioning: MAJOR.MINOR.PATCH
# v1.0.0, v1.1.0, v1.1.1

# Create annotated tag
git tag -a v1.2.0 -m "Release version 1.2.0"

# Push tags
git push origin v1.2.0
git push origin --tags

# List tags
git tag -l

# Checkout specific tag
git checkout v1.2.0
```

## 🚨 Self-Enforcement — WAJIB di Setiap Git Operation

**Sebelum melakukan git operations (commit, branch, push, PR), WAJIB cek:**
1. Apakah branch saat ini sudah benar sesuai peruntukannya? (JANGAN commit ke `main`/`master` langsung).
2. Apakah commit message sudah mengikuti standar Conventional Commits (feat, fix, docs, refactor, dll)?
3. Apakah ada file sensitif (.env, password, API keys) yang tidak sengaja ter-stage? Cek `.gitignore`.
4. Apakah ada print debug, console.log, atau commented-out code yang lupa dihapus sebelum commit?
5. Apakah test locally sudah pass sebelum pushing ke remote?

**JIKA ada aturan yang dilanggar → koreksi SEBELUM lanjut ke task lain.**

---

## Git Workflow Checklist

```
Before Starting Work
□ Pull latest changes dari develop
□ Create feature branch dengan naming convention
□ Branch dari develop (bukan main)

During Development
□ Commit frequently dengan clear messages
□ Follow commit message standards
□ Keep commits atomic (one logical change)
□ Pull & rebase develop regularly

Before Creating PR
□ Self-review changes
□ Run tests locally
□ Update documentation
□ Rebase & squash commits jika perlu
□ Push to remote

Pull Request
□ Clear title & description
□ Link related issues
□ Request reviewers
□ Respond to review comments
□ Update PR based on feedback

After Merge
□ Delete feature branch
□ Pull latest develop
□ Celebrate! 🎉
```
