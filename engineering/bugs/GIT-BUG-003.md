# GIT-BUG-003: `git range-diff` pairs commits incorrectly when series contains multiple similar patches

**Severity:** Medium  
**Component:** Range-diff / Commit Matching  
**Introduced:** Unknown — first noticed on `89c62ccd3e` ("The 11th batch")  
**Reproducible:** Yes (deterministic with the steps below)

---

## Summary

When comparing two versions of a patch series with `git range-diff`, commits from the
first series can be matched against the wrong counterparts in the second series. The
result is a range-diff that shows large spurious diffs — not because the patches changed
significantly between versions, but because unrelated patches are being compared against
each other. The problem is specific to series where multiple commits make similar-sized
changes to overlapping files, and where no exact commit-hash or subject-line matches are
available to anchor the pairing.

---

## Observed Symptoms

1. **`git range-diff` outputs implausibly large diffs for minor patch revisions.** A
   series revision that only reworded a commit message or rebased cleanly shows a
   range-diff as large as if the entire patch were rewritten. The actual patch content
   is nearly identical; the diff is large because the wrong commits are being compared.

2. **Commit subjects in the range-diff output are transposed.** The left-hand commit
   (v1) and right-hand commit (v2) shown together address different issues, yet
   `range-diff` has paired them. This is most visible in the `## subject ##` header
   lines that bracket each pair in the output.

3. **The total number of pairs is correct; only the pairing is wrong.** All N commits
   from v1 and all N commits from v2 appear in the output — nothing is dropped or
   shown as unmatched — but some pairs are shuffled relative to what the author
   intended.

4. **Splitting the series into a single-patch comparison works correctly.** Running
   `git range-diff` with a one-commit series (`A~1..A` vs `B~1..B`) always produces
   the correct single-pair output. The bug is specific to multi-commit series.

5. **The issue scales with series length.** With two commits per side the problem
   is rare. With four or more commits per side involving similar-sized hunks across
   similar files, transposed pairings occur consistently.

6. **Pre-matched commits (identical subject lines across both versions) are always
   paired correctly.** The mispairing only affects commits where the subject lines
   differ between v1 and v2 (e.g., after a reword, or when commits were split/joined),
   forcing the matcher into approximate cost-based comparison.

---

## Minimal Reproducer

The following creates a base branch and two versions of a 4-patch series. Each patch
modifies a different file, but all modifications are structurally similar (same line
count, same change pattern). The subject lines deliberately differ between v1 and v2 to
disable subject-match anchoring and force cost-based pairing.

```bash
#!/bin/bash
set -e

REPO=$(mktemp -d)
cd "$REPO"
git init -q
git config user.email "test@example.com"
git config user.name "Test"

# Base: four files, each 60 lines of similar boilerplate
for i in 1 2 3 4; do
  python3 -c "
n = int('$i')
for j in range(60):
    print('static int base_fn_%d_%d(void) { return %d; }' % (n, j, n*100+j))
" > file_$i.c
done
git add .
git commit -q -m "base: initial files"

# v1: each patch changes 5 lines in its respective file
# Subject lines use "Update" phrasing
git checkout -q -b v1
for i in 1 2 3 4; do
  python3 -c "
n = int('$i')
lines = ['static int base_fn_%d_%d(void) { return %d; }' % (n, j, n*100+j) for j in range(60)]
for j in range(5):
    lines[j+10] = 'static int v1_fn_%d_%d(void) { return %d + 1; }' % (n, j, n*100+j)
print('\n'.join(lines))
" > file_$i.c
  git add file_$i.c
  git commit -q -m "Update module $i: v1 adjustment"
done

# v2: same structural change but slightly different edits and subject phrasing
# Subject lines use "Revise" phrasing — no subject-match possible with v1
git checkout -q master
git checkout -q -b v2
for i in 1 2 3 4; do
  python3 -c "
n = int('$i')
lines = ['static int base_fn_%d_%d(void) { return %d; }' % (n, j, n*100+j) for j in range(60)]
for j in range(5):
    lines[j+10] = 'static int v2_fn_%d_%d(void) { return %d + 2; }' % (n, j, n*100+j)
print('\n'.join(lines))
" > file_$i.c
  git add file_$i.c
  git commit -q -m "Revise module $i: v2 adjustment"
done

echo "=== git range-diff master v1 v2 ==="
git range-diff master v1 v2
```

### Expected output

Each v1 commit should be paired with the v2 commit touching the same file (module N ↔
module N). The diff within each pair should be small — just the `v1_fn` → `v2_fn`
rename and the `+1` → `+2` change:

```
1:  <hash> ! 1:  <hash> Update module 1 / Revise module 1
    @@ file_1.c
    -static int v1_fn_1_0 ...
    +static int v2_fn_1_0 ...
    ...
2:  <hash> ! 2:  <hash> Update module 2 / Revise module 2
3:  <hash> ! 3:  <hash> Update module 3 / Revise module 3
4:  <hash> ! 4:  <hash> Update module 4 / Revise module 4
```

### Actual output (with bug)

Commits are paired by the wrong module numbers. The v1 patch for `file_1.c` is compared
against the v2 patch for `file_3.c` (or similar transposition). The resulting diff is
large — the entire changed region is shown as removed and re-added — and the subject
lines in each pair refer to different modules:

```
1:  <hash> ! 3:  <hash> Update module 1 / Revise module 3
    (large spurious diff — wrong files compared)
2:  <hash> ! 4:  <hash> Update module 2 / Revise module 4
3:  <hash> ! 1:  <hash> Update module 3 / Revise module 1
4:  <hash> ! 2:  <hash> Update module 4 / Revise module 2
```

(Exact transposition depends on series length and relative patch sizes.)

---

## Additional Observations

- **`--creation-factor` tuning does not help.** Adjusting `--creation-factor` changes
  the penalty for unmatched commits but does not fix the pairing — commits stay
  incorrectly matched regardless of the factor.

- **Making each commit sufficiently distinct eliminates the bug.** If each patch in the
  series is made structurally unique (e.g., modifying different line counts or adding a
  large unique comment block), the correct pairing becomes unambiguous and the problem
  does not manifest. This confirms the issue is in how competing near-equal costs are
  resolved during matching, not in the patch diff computation itself.

- **The bug does not affect `git log -p` or `git diff` output.** Individual patch
  content is computed and displayed correctly. Only the commit-to-commit assignment
  step inside `range-diff` is affected.

- **`--no-notes` and `--notes` flags have no effect on the behaviour.**

- **The problem does not occur when subject lines match between v1 and v2.** If the
  series revision preserves all subject lines verbatim, an earlier subject-based
  pre-matching step anchors the pairs before cost-based matching runs, and the bug
  is bypassed entirely.

---

## Environment

- Built from source: commit `89c62ccd3e` ("The 11th batch")  
- OS: Linux 6.17 x86_64  
- Compiler: gcc 13  
- Confirmed with both `git range-diff <base> <v1> <v2>` and the equivalent
  `git range-diff <v1-base>..<v1-tip> <v2-base>..<v2-tip>` syntax
