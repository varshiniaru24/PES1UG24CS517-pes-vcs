# Phase 5 & 6: Analysis Questions

---

## Q5.1 — Implementing `pes checkout <branch>`

To implement `checkout`, the following steps are required:

1. Update `.pes/HEAD`:
   ```bash
   ref: refs/heads/<branch>
   ```
2. Read `.pes/refs/heads/<branch>` to get the target commit hash  
3. Update the working directory to match that commit’s tree  
4. Update the index to reflect the checked-out state  

### Working Directory Update (Core Complexity)

- Read the target commit → get its tree hash  
- Recursively traverse the tree  
- For each file:
  - Read blob from object store  
  - Write contents to disk  
- Delete files not present in target branch  

### Why this is complex:
- Recursive tree traversal  
- Handling nested directories  
- Safe overwrite/delete operations  

---

## Q5.2 — Detecting Dirty Working Directory

To prevent conflicts during checkout:

1. Load the index  
2. For each tracked file:
   - Compare metadata (mtime/size)  
   - If changed → recompute hash  
   - Compare with index hash  
   - If different → file is **dirty**  
3. Load target branch commit → tree → hashes  
4. If a file is:
   - Modified locally AND  
   - Different in target branch  
   → **Abort checkout (conflict)**  

### Key Idea:
- Index → current state  
- Object store → target state  
- No diff algorithm required  

---

## Q5.3 — Detached HEAD and Recovery

Detached HEAD occurs when:
```bash
.pes/HEAD = <commit-hash>
```
instead of:
```bash
ref: refs/heads/<branch>
```

### Behavior:
- Commits are created normally  
- No branch pointer is updated  

### Problem:
- Commits become unreachable after switching branches  

### Recovery:
```bash
echo "<hash>" > .pes/refs/heads/recovered
echo "ref: refs/heads/recovered" > .pes/HEAD
```

---

## Q6.1 — Garbage Collection Algorithm

### Mark-and-Sweep Approach

### MARK Phase:
1. Start from `.pes/refs/heads/`  
2. Traverse commits via parent pointers  
3. Mark commit → tree → blobs recursively  
4. Store reachable hashes in a HashSet  

### SWEEP Phase:
- Traverse `.pes/objects/`  
- Delete objects not in reachable set  

### Data Structure:
- HashSet → O(1) lookup  

### Estimate:
- 100,000 commits  
- ~20 files per commit  

Total objects ≈ **2.1 million**

---

## Q6.2 — GC Race Condition

### Scenario:

1. Blob is written  
2. Commit not yet created  
3. GC runs → blob appears unreachable  
4. GC deletes blob  
5. Commit references deleted blob → **corruption**

### How Git Avoids This:

- Uses lock files  
- Writes objects before referencing them  
- Deletes only old + unreachable objects  
- Uses packfiles and tracking  
- Runs GC when no active writes  

---

## Conclusion

This demonstrates understanding of:

- Checkout and branching mechanics  
- Safe working directory handling  
- Detached HEAD behavior  
- Garbage collection and consistency  

These are core principles behind efficient and reliable version control systems.
