## Phase 5 & 6: Analysis Questions

### Q5.1 — Implementing pes checkout <branch>

To implement checkout, three things must happen:
1. `.pes/HEAD` must be updated to `ref: refs/heads/<branch>`
2. The branch ref file `.pes/refs/heads/<branch>` must be read to get the commit hash
3. The working directory must be updated to match the tree of that commit

The complexity comes from step 3. You must:
- Read the target commit → get its tree hash
- Recursively walk the tree to get all file paths and blob hashes
- For each file: read the blob from the object store and write it to disk
- Delete files that exist in the current branch but not the target branch

This is complex because it requires recursive tree traversal, careful file deletion,
and handling of nested directory structures.

### Q5.2 — Detecting Dirty Working Directory

To detect conflicts before switching branches:
1. Load the index (which records the last-staged hash and mtime/size of each file)
2. For each tracked file, compare current disk state to the index using stat():
   - If mtime or size differs → file is modified (dirty)
3. For the target branch, read its commit → tree → get blob hashes
4. If a dirty file has a different blob hash between branches → conflict → refuse checkout

This works purely using the index (for current state) and the object store
(for target branch state), with no need for any diff algorithm.

### Q5.3 — Detached HEAD and Recovery

In detached HEAD state, HEAD contains a raw commit hash instead of
`ref: refs/heads/branch`. New commits are created and chained normally,
but no branch pointer is updated to track them.

If you switch away, these commits become unreachable — no branch points to them.
To recover: use `pes log` output (if still visible in terminal) to find the
commit hash, then create a new branch pointing to it:
  echo "<hash>" > .pes/refs/heads/recovery
  echo "ref: refs/heads/recovery" > .pes/HEAD

### Q6.1 — Garbage Collection Algorithm

Algorithm (Mark and Sweep):
1. MARK phase: Start from all branch refs in `.pes/refs/heads/`
2. For each branch, walk the commit chain (following parent pointers)
3. For each commit, mark its tree hash as reachable
4. Recursively walk each tree, marking all sub-tree and blob hashes
5. Add all visited hashes to a HashSet
6. SWEEP phase: Walk all files in `.pes/objects/` — delete any whose
   hash is NOT in the HashSet

Data structure: A hash set (e.g., a hash table or sorted array of 64-char strings)
gives O(1) lookup for the reachability check.

Estimate for 100,000 commits and 50 branches:
- 100,000 commits × avg ~20 files each = ~2,000,000 blob/tree objects to visit
- Total objects visited ≈ 2,100,000 (commits + trees + blobs)

### Q6.2 — GC Race Condition

Race condition scenario:
1. A commit operation creates a new blob object and writes it to the object store
2. Before the commit object referencing that blob is written, GC runs
3. GC scans all refs → the new blob is not yet reachable (no commit points to it)
4. GC deletes the blob
5. The commit finishes writing its commit object referencing the now-deleted blob
6. The repository is now corrupt

How Git avoids this:
- Git uses a "grace period" — objects newer than 2 weeks are never deleted by GC
- This gives in-progress operations time to complete before GC considers objects
- Git also writes a `MERGE_HEAD` / lock file during operations so GC can detect
  concurrent writes and skip or wait
