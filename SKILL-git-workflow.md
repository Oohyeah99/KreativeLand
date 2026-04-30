# Git Workflow Skill

## When to Use
- Making commits to any project in the ecosystem
- Pushing to GitHub
- Setting up a new git repository
- Fixing git issues (especially OneDrive-related)
- Handling authentication or GFW push failures

## CRITICAL: External .git Directory Pattern

**Problem:** OneDrive corrupts/deletes `.git` directories. This has destroyed git repos multiple times.

**Solution:** All `.git` directories are stored at `D:\GitRepos\`. Project folders have a `.git` **file** (not directory) pointing to the external location.

### Project Registry

| Project | .git file content | GitHub Remote |
|---------|------------------|---------------|
| VocabVista | `gitdir: D:/GitRepos/VocabVista.git` | https://github.com/Oohyeah99/VocabVista.git |
| EnglishExamPrep | `gitdir: D:/GitRepos/EnglishExamPrep.git` | https://github.com/Oohyeah99/EnglishExamPrep.git |
| ZhongkaoPrep | `gitdir: D:/GitRepos/ZhongkaoPrep.git` | https://github.com/Oohyeah99/ZhongkaoPrep.git |
| LittleReader | `gitdir: D:/GitRepos/LittleReader.git` | https://github.com/Oohyeah99/LittleReader.git |
| KreativeLand | `gitdir: D:/GitRepos/KreativeLand.git` | https://github.com/Oohyeah99/KreativeLand.git |
| LittleLearner | `gitdir: D:/GitRepos/LittleLearner.git` | https://github.com/Oohyeah99/LittleLearner.git |
| AIPlayground | `gitdir: D:/GitRepos/AIPlayground.git` | https://github.com/Oohyeah99/AIPlayground.git |

### Rules
1. **NEVER delete or modify the `.git` file** in the project folder
2. **If git stops working** ("not a git repository"), check if the `.git` file exists and has the correct `gitdir:` path. Recreate if OneDrive deleted it
3. **To init a new project:**
   ```bash
   git init --separate-git-dir="D:/GitRepos/ProjectName.git"
   git remote add origin https://github.com/Oohyeah99/ProjectName.git
   ```

## Git Identity
Use this for ALL commits across the ecosystem:
- **Name:** `KL`
- **Email:** `kl@kreativeland.com`

Set per-repo (already configured):
```bash
git config user.name "KL"
git config user.email "kl@kreativeland.com"
```

## Pushing to GitHub

### ALWAYS Use HTTP/1.1
The Great Firewall (GFW) blocks HTTP/2 to GitHub:
```bash
git -c http.version=HTTP/1.1 -c http.postBuffer=524288000 push origin master
```

### Credential Handling
- Global config has `credential.guiPrompt=false` and `credential.interactive=never`
- Credentials cached by Git Credential Manager (GCM)
- If push fails with auth error, retry -- GCM will refresh the OAuth token silently

### GFW Push Failures
If push fails with `connection reset` or `timeout`:
1. Tell KL: "Push failed -- likely GFW. Can you check your proxy is on?"
2. Wait for confirmation, then retry
3. Do NOT silently retry more than once

### Commit Best Practices
- Keep commits focused on a single change
- Use descriptive commit messages
- Don't include secrets/credentials (GitHub secret scanning will reject them)
- Use HEREDOC for complex messages to avoid shell escaping:
  ```bash
  git commit -m "$(cat <<'EOF'
  Fix: Updated nginx config for api.kreativeland.com
  
  Removed ai-playground symlink from sites-enabled
  that was catching traffic for unknown domains.
  EOF
  )"
  ```

## Troubleshooting

### "not a git repository"
Check if `.git` file exists:
```bash
ls -la .git
```
If missing, recreate:
```bash
echo "gitdir: D:/GitRepos/ProjectName.git" > .git
```

### Push rejected (secret scanning)
GitHub detected a secret in your commit. Remove the secret from the file, amend the commit:
```bash
# Fix the file, then:
git add <file> && git commit --amend --no-edit
```

### Branch tracking lost
```bash
git branch --set-upstream-to=origin/master master
```
