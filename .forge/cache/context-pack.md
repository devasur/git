# Architecture Context Pack

Built: 2026-06-07T19:13:01.242Z
Source hash: sha256:d967a5bb00bf85cc30f662f9f05746f19f84ebcef5459a613e5b92a72d320371

## Key Data Structures

| Struct | File | Purpose |
|--------|------|---------|
| `struct object_id` | `hash.h` / `object-name.h` | Hash value (SHA-1 20B or SHA-256 32B) |
| `struct commit` | `commit.h` | Commit object with parent list, tree OID |
| `struct tree` | `tree.h` | Tree object with entry list |
| `struct blob` | `blob.h` | Blob object (content loaded on demand) |
| `struct ref_store` | `refs-internal.h` | Polymorphic ref backend interface |
| `struct packed_git` | `pack.h` | In-memory packfile descriptor |
| `struct object_database` [?] | `odb.h` | Object store abstraction |

## Document summaries

### Architecture — Index

| Document | Description | |----------|-------------| | [stack.md](stack.md) | Languages, frameworks, runtime, versions, dependencies | | [processes.md](processes.md) | Services, build/deploy topology, CI/CD pipeline | | [database.md](database.md) | Object storage model, ref backends, key data structures | | [routing.md](routing.md) | Command dispatch, protocol handling, auth strategy | | [deployment.md](deployment.md) | Build configurations, CI workflows, installation, release process |

### Database / Object Storage

Git does not use a relational database. It implements its own content-addressable object store on the filesystem, commonly called the **object database (ODB)**.

### Deployment

### Entity Model

Git's domain model is a **content-addressable object graph** rather than a relational schema. The entities below correspond to the four Git object types plus the ref and index structures.

### Processes

### Routing / Command Dispatch

Git is not a web application. Its "routing" is **command dispatch**: the `git` binary resolves a subcommand name to a built-in handler or an external executable.

### Stack Review Checklist

Use this checklist when reviewing code changes in the git codebase.

### Stack

## File index

- engineering/architecture/INDEX.md — Architecture — Index (11 lines)
- engineering/architecture/database.md — Database / Object Storage (68 lines)
- engineering/architecture/deployment.md — Deployment (71 lines)
- engineering/architecture/entity-model.md — Entity Model (135 lines)
- engineering/architecture/processes.md — Processes (70 lines)
- engineering/architecture/routing.md — Routing / Command Dispatch (70 lines)
- engineering/architecture/stack-checklist.md — Stack Review Checklist (71 lines)
- engineering/architecture/stack.md — Stack (53 lines)
