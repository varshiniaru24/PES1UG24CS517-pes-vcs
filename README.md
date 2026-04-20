# PES-VCS — A Version Control System from Scratch

**Name:** Varshini A  
**SRN:** PES1UG24CS517  
**Platform:** Ubuntu 22.04  

---

## Overview

PES-VCS is a simplified version control system built to understand the internal working of systems like Git. It demonstrates how files are stored, tracked, and versioned using concepts such as hashing, trees, staging, and commits.

The project is divided into multiple phases, each implementing a core component of a version control system.

---

## Phase 1 — Object Storage System

This phase implements the foundation of storage using a content-addressable system.

### Features
- Files stored as immutable blob objects
- SHA-256 hashing for unique identification
- Deduplication of identical file contents
- Sharded storage inside `.pes/objects/`

### Key Functions
- `object_write()` → stores file content as an object
- `object_read()` → retrieves and verifies object integrity

### Outcome
This phase ensures data integrity and avoids duplicate storage of identical files.

---

## Phase 2 — Tree Structure

This phase introduces hierarchical representation of directories.

### Features
- Directory structure stored as tree objects
- Recursive representation of folders
- Mapping between filenames and object hashes

### Key Function
- `tree_from_index()` → builds tree objects from staged files

### Outcome
Directories are represented logically rather than physically, enabling snapshot reconstruction.

---

## Phase 3 — Index (Staging Area)

This phase implements the staging area between working directory and repository.

### Features
- Tracks files selected for commit
- Stores metadata (hash, size, timestamp, path)
- Atomic updates to index file

### Key Functions
- `index_load()` → loads existing staged state
- `index_save()` → writes index safely
- `index_add()` → stages files into index

### Outcome
Only staged changes are included in commits, providing controlled versioning.

---

## Phase 4 — Commit System

This phase builds the version history system.

### Features
- Snapshot-based commits
- Parent commit linking
- HEAD and branch updates

### Key Function
- `commit_create()` → creates commit from index state

### Outcome
Each commit represents a full snapshot of the project and forms a linked history chain.

---

## Phase 5 — Branching and Checkout (Analysis)

### Checkout Process
- HEAD is updated to point to selected branch
- Commit hash is resolved from branch reference
- Working directory is reconstructed from commit tree
- Index is updated accordingly

### Dirty Working Directory Check
- Working directory is compared with index state
- File modifications detected using metadata
- Checkout is blocked if conflicts exist

### Detached HEAD
- HEAD points directly to a commit instead of a branch
- New commits are created but not attached to any branch
- Such commits can be recovered by creating a new branch

---

## Phase 6 — Garbage Collection (Analysis)

### Reachability Algorithm
- Start from branch references
- Traverse commit history via parent links
- Mark reachable trees and blobs
- Delete unmarked objects

### Race Condition Issue
If garbage collection runs during active commit creation:
- Newly created objects may be deleted prematurely
- Leads to repository corruption

### Prevention
- Locking during writes
- GC runs only on stable repository state
- Ensures consistency of object store

---

## Conclusion

PES-VCS demonstrates how a version control system can be built from basic filesystem and hashing concepts. The project provides hands-on understanding of:

- Content-addressable storage
- Tree-based directory representation
- Staging and commit workflow
- Branching and HEAD mechanics
- Garbage collection principles

This system closely mirrors the internal architecture of real-world version control tools like Git.
