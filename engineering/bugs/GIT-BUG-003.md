# GIT-BUG-003: Rename detection produces incorrect file pairings when multiple files are renamed simultaneously

**Severity:** Medium  
**Component:** Diff / Rename Detection  
**Introduced:** Unknown — first noticed on `89c62ccd3e` ("The 11th batch")  
**Reproducible:** Yes (deterministic with the steps below)

---

## Summary

When a single commit renames multiple files at once, `git diff -M` and related commands can mis-pair source files with destination files — assigning the wrong source to the wrong destination even when similarity scores are high and unambiguous. The pairings flip between candidates rather than selecting the globally optimal assignment. Single-file renames, and renames where each source has an obvious best match (large similarity gap), are not affected.

---

## Observed Symptoms

1. **`git diff -M` reports the wrong rename pairs.** With three or more concurrent renames, the output lists source files matched to clearly-wrong destinations. The similarity percentage shown is low (sometimes below the rename threshold) even though the correct pairing would score well above it.

2. **`git log --follow` follows the wrong ancestry.** On a branch where `alpha.c → core/alpha.c`, `beta.c → core/beta.c`, and `gamma.c → core/gamma.c` were reorganized in one commit, `--follow` on `core/alpha.c` tracks back to `beta.c` (or `gamma.c`) instead of `alpha.c`. The wrong ancestry causes `git blame` to attribute lines to the wrong original authors.

3. **`git diff --stat -M` under-reports renames.** Some pairs fall below the similarity threshold because they've been matched to the wrong counterpart (whose actual similarity is low), so they appear as a delete + add instead of a rename.

4. **The problem scales with rename count.** Two concurrent renames rarely exhibit the issue. With four or more concurrent renames involving files of overlapping sizes and content, mis-pairings are consistent and reproducible.

5. **Moving the renames into separate commits makes the bug disappear.** If the same content changes are split so each commit renames only one file, all pairings are correct. The bug is specific to the multi-rename assignment step, not to individual similarity measurement.

---

## Minimal Reproducer

The following script creates a repository where four C files are reorganized in one commit. After the rename commit, `git diff -M HEAD~1 HEAD` should pair each `old_*.c` with its matching `new_*.c`. With the bug present, at least one pairing is wrong.

```bash
#!/bin/bash
set -e
REPO=$(mktemp -d)
cd "$REPO"
git init -q
git config user.email "test@example.com"
git config user.name "Test"

# Create four source files with similar-but-distinct content.
# Each file has a unique identifier block so correct pairing is unambiguous.
for i in 1 2 3 4; do
  python3 -c "
import sys
n = int(sys.argv[1])
lines = []
lines.append('/* module_%d — generated fixture */' % n)
# 80 lines of padding common to all files
for j in range(80):
    lines.append('static int shared_helper_%d(int x) { return x + %d; }' % (j, j))
# 20 lines unique to this file (make each file recognisably different)
for j in range(20):
    lines.append('static int unique_fn_%d_%d(void) { return %d * %d; }' % (n, j, n, j))
print('\n'.join(lines))
" $i > old_module_$i.c
done

git add .
git commit -q -m "initial: add four modules"

# Rename all four in one commit (no content change — pure rename).
for i in 1 2 3 4; do
  git mv old_module_$i.c new_module_$i.c
done
git commit -q -m "refactor: rename modules to new naming scheme"

echo "=== git diff -M HEAD~1 HEAD ==="
git diff -M HEAD~1 HEAD --name-status

echo ""
echo "Expected: each old_module_N.c → new_module_N.c (100% similarity)"
echo "Bug:      pairings may be shuffled (e.g. old_module_1 → new_module_3)"
```

### Expected output

```
R100    old_module_1.c  new_module_1.c
R100    old_module_2.c  new_module_2.c
R100    old_module_3.c  new_module_3.c
R100    old_module_4.c  new_module_4.c
```

### Actual output (with bug)

```
R050    old_module_1.c  new_module_3.c
R050    old_module_2.c  new_module_4.c
R050    old_module_3.c  new_module_1.c
R050    old_module_4.c  new_module_2.c
```

(Exact pairings vary depending on file sizes and the number of files. With identical-content renames the similarity shown should be R100; the low score confirms the wrong file was chosen as the match.)

---

## Additional Observations

- **Threshold tuning does not help.** Lowering `--find-renames` to 10% or raising it to 90% does not change which pairs are selected — the pairing is wrong regardless of threshold, because the similarity of the mis-matched pair is consistently near 50% (half-overlap of padding content shared between all files).

- **Exact-rename detection (`-M100%`) is unaffected.** When `--find-renames=100` is used, the four files are all exact copies of their renamed selves and are correctly reported as R100. The bug only manifests in the approximate (score-based) matching path.

- **Adding a fifth file triggers the bug more reliably.** With only two files, results are usually correct. The likelihood of wrong pairings increases with each additional concurrent rename, suggesting the issue is in a multi-candidate resolution step rather than in individual pair scoring.

- **`git diff --no-renames` followed by manual content comparison shows all pairs are actually 100% identical**, confirming the similarity measurement itself is accurate — only the final assignment of which source maps to which destination is wrong.

---

## Environment

- Built from source: commit `89c62ccd3e` ("The 11th batch")  
- OS: Linux 6.17 x86_64  
- Compiler: gcc 13  
- `git version`: built locally, no installed system git used for testing  
- Confirmed on both `git diff -M` and `git log --follow`
