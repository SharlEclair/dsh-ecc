# Fork and PR

`gh` 2.102.0 is installed. Authentication is the one step that needs you.

## 1. Authenticate (once, interactive)

```powershell
gh auth login
```

Choose **GitHub.com** → **HTTPS** → **Login with a web browser**, and complete it in the browser.
Verify:

```powershell
gh auth status
```

## 2. Fork and rewire remotes

The clone's `origin` currently points at the upstream project. After forking, `origin` becomes
**your** fork and upstream moves to a second remote:

```powershell
Set-Location C:\Users\91704\Desktop\dsh-ecc

gh repo fork --remote --remote-name origin
# Re-point the original at upstream
git remote set-url origin https://github.com/<YOUR-USER>/dsh-ecc.git
git remote add upstream https://github.com/gongyijie85/dsh-ecc.git
git remote -v
```

Expected: `origin` → your fork, `upstream` → `gongyijie85/dsh-ecc`.

## 3. Branch per adaptation batch

Never work on `main` — it must stay fast-forwardable to upstream.

```powershell
git checkout -b fix/tier-c-ecc-cli-refs
```

## 4. Commit and push

```powershell
npm run verify                       # must stay green
git add -A
git commit -m "fix(configure-ecc,security-scan,unified-memory): replace ecc CLI references with DSH-native equivalents"
git push -u origin fix/tier-c-ecc-cli-refs
```

## 5. Open the PR

```powershell
gh pr create --repo gongyijie85/dsh-ecc --fill
```

Or push and click **Compare & pull request** on github.com.

### PR body checklist

State plainly, because the maintainer will check:

- [ ] Which skills changed and **why each was wrong** for DSH
- [ ] `npm run verify` output (paste the `OK: all 273` line)
- [ ] That `scripts/verify-provider.mjs` was updated if the skill set changed
- [ ] DSH version tested against (this workspace: `0.2.0-rc.2`)
- [ ] Whether you ran it inside a live DSH session, or only the standalone verifier

Do not claim runtime verification you did not perform.

## Keeping in sync with upstream

```powershell
git fetch upstream
git checkout main
git merge --ff-only upstream/main      # fails loudly if you committed to main — that is the point
git push origin main
```

If `--ff-only` fails, you have commits on `main`. Move them to a branch rather than forcing.

## Caution

- **Do not** `git push --force` to `main` on either remote.
- **Do not** open one giant PR adapting all ~40 files. Small, reviewable PRs land; a 40-file
  diff will not. Recommended: one PR per tier.
- Upstream's GitHub `main` is `0.6.0` (this clone matches it), while npm carries `0.6.1` with no
  matching git tag. Rebase on `upstream/main` before opening a PR so you are not asking to merge
  against a stale base.
