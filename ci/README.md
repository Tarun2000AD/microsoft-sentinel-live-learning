# CI

[`structure-check.yml`](structure-check.yml) is a GitHub Actions workflow that verifies:

- every `NN-*` folder has a `README.md`
- steps are numbered `00`–`62` with no gaps or duplicates
- no secrets or Logic App callback URLs are committed

**To enable it**, move the file into place (it lives here because the initial push
was made with a token lacking the `workflow` scope):

```bash
mkdir -p .github/workflows
git mv ci/structure-check.yml .github/workflows/structure.yml
git commit -m "Enable structure-check workflow"
git push
```

You can also run the checks locally without Actions:

```bash
for d in [0-9][0-9]-*/; do [ -f "${d}README.md" ] || echo "missing: $d"; done
diff <(ls -d [0-9][0-9]-*/ | sed -E 's/^([0-9]{2}).*/\1/') <(seq -w 0 62)
```
