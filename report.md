# Building PES-VCS — A Version Control System from Scratch

**Name:** Varshini A
**SRN:** PES1UG24CS517  
**Platform:** Ubuntu 22.04  

---

## Objective

The goal of this project was to build a simplified version control system (PES-VCS) from scratch. The system behaves similarly to Git internally and helped us understand how commits, trees, blobs, and indexing actually work at the filesystem level.

We implemented object storage, tree structures, staging area (index), commit system, and analyzed branching + garbage collection concepts.

---

## Phase 1: Object Storage System

In this phase, I implemented the core idea of content-addressable storage. Every file is stored as a blob using its SHA-256 hash.

### What I learned
- How Git avoids duplicate storage using hashing
- How objects are stored in `.pes/objects/`
- How integrity checking works

### Key Implementation
- `object_write()` stores file content with SHA-256 hash
- `object_read()` retrieves and verifies integrity

### Observations
Even if the same file is added multiple times, it is stored only once because the hash remains the same.

📸 **1A:** Test output showing successful object creation and retrieval  
📸 **1B:** `.pes/objects` directory showing sharded structure

---

## Phase 2: Tree Structure

This phase focused on building directory structures using tree objects.

### What I learned
- How folders are represented internally
- How recursion is used to build nested structures
- How trees connect blobs and subtrees

### Key Implementation
- `tree_from_index()` recursively builds tree objects from index entries

### Observations
A directory is not stored as a real folder but as a structured object containing references.

📸 **2A:** Tree test passing output  
📸 **2B:** Hex dump of a tree object showing binary structure

---

## Phase 3: Index (Staging Area)

This was one of the most important parts — implementing the staging area like Git’s index.

### What I learned
- Difference between working directory, index, and repository
- How file metadata helps detect changes
- How staging prepares commits

### Key Implementation
- `index_load()` reads staged files
- `index_save()` writes updates atomically
- `index_add()` stages a file into the system

### Observations
The index acts as a bridge between working files and commits.

📸 **3A:** `pes add` and `pes status` output  
📸 **3B:** `.pes/index` file content

---

## Phase 4: Commit System

This phase brought everything together.

### What I learned
- How commits point to trees
- How parent commits form history chains
- How HEAD updates work internally

### Key Implementation
- `commit_create()` builds commit object from index + tree
- Updates branch reference automatically

### Observations
Each commit represents a full snapshot, not a difference.

📸 **4A:** `pes log` showing commit history  
📸 **4B:** Object store growth after commits  
📸 **4C:** HEAD and branch reference files

---

## Phase 5 & 6: Analysis Questions

### Q5.1 — Checkout Implementation

To implement `pes checkout <branch>`, the system must:
- Update `.pes/HEAD` to point to the branch reference
- Read commit hash from `.pes/refs/heads/<branch>`
- Load commit → extract tree → restore files in working directory

The hardest part is restoring files correctly:
- Trees must be traversed recursively
- Old files not in new branch must be deleted
- File structure must be rebuilt exactly

---

### Q5.2 — Dirty Working Directory Detection

To prevent data loss during checkout:
- Compare working directory files with index metadata
- Use `stat()` to detect changes in mtime/size
- Compare with target branch’s tree blobs
- If mismatch exists → refuse checkout

This avoids overwriting uncommitted changes.

---

### Q5.3 — Detached HEAD

In detached HEAD state:
- HEAD points directly to a commit, not a branch
- New commits are created but not linked to any branch
- If you switch branches, these commits can become unreachable

**Recovery method:**
Create a new branch pointing to the lost commit hash.

---

### Q6.1 — Garbage Collection

Garbage collection removes unreachable objects.

### Algorithm:
1. Start from all branch heads
2. Traverse commits → trees → blobs
3. Mark all reachable objects
4. Delete unmarked objects from `.pes/objects`

### Data Structure:
- HashSet for storing visited object hashes

### Complexity:
For 100,000 commits and 50 branches:
- Millions of objects may need traversal (~2M range)

---

### Q6.2 — GC Race Condition

If GC runs during commit:
- New object is created but not yet referenced
- GC may incorrectly mark it as unreachable
- Object gets deleted before commit finishes

This causes repository corruption.

### How Git solves it:
- Uses locking during commit
- Ignores very new objects (grace period)
- Prevents GC during active writes

---

## Conclusion

This project helped me understand how Git works internally at a very deep level. I now understand:

- How files are stored using hashing
- How commits represent snapshots
- How trees simulate directories
- How staging area connects everything
- How garbage collection and branching work internally

Overall, PES-VCS is a simplified but powerful model of real-world version control systems.

---

## Screenshots Summary

- 1A, 1B → Object storage
- 2A, 2B → Tree structure
- 3A, 3B → Index/staging area
- 4A, 4B, 4C → Commits + HEAD

---

## Final Note

All phases were implemented with proper commits and incremental development. Each phase contains at least 5 commits showing step-by-step progress as required.
