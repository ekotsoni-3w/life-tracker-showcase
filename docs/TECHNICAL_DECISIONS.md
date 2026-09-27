# Technical decisions

## Central task aggregation without duplication

Several modules produce actionable items: personal tasks, office and teaching plans, schoolwork, shopping, and planned financial activity. The centralized To Do view reads and projects these records into a shared shape. Each projected item carries its source and path, so completing it writes back to the original module.

This favors consistency over a second task database. The tradeoff is that every source adapter must validate its own schema and expose a predictable projection.

## Storage abstraction

Page code uses a shared key/value storage interface:

- Guest mode delegates to browser storage.
- Account mode reads from an in-memory account snapshot.
- Writes first update a recoverable local pending snapshot, then enter a serialized server synchronization loop.

The abstraction lets product modules behave consistently across both modes and keeps network concerns out of page components.

## Revision-based conflict handling

Account snapshots include a version. Updates send the version they were based on; a stale version is reported as a conflict rather than overwriting newer remote data. A pending local copy remains available for export or deliberate recovery.

This is intentionally explicit. Automatic record-level merging was not chosen because modules contain different nested schemas and an invisible merge could corrupt user intent.

## Integer cents for money

Budget values are parsed once and stored as integer cents. Totals remain integers until display formatting. Validation also applies upper bounds and safe-integer checks.

This avoids floating-point rounding errors and makes financial totals deterministic.

## Defensive parsing

Every persisted module validates structure, IDs, field types, supported enum values, and relevant date/number constraints on load. A malformed collection is blocked from mutation and the UI offers a retry path.

The goal is to avoid turning one malformed record into a silently rewritten dataset.

## Authentication and ownership boundaries

The account backend uses salted scrypt password hashes, HttpOnly sessions, production secure-cookie settings, email verification, hashed one-time reset tokens, and ownership checks on account-scoped operations and files.

These controls are implementation measures, not a substitute for an independent security audit, operational monitoring, or a production-readiness review.

