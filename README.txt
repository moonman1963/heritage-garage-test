HERITAGE PROTOTYPE — CLEAN REBUILD

Branch: rebuild-v0.1
Backend: Supabase development branch rebuild-v0-1

Current build status: v0.4 hardening in progress

Scope:
- Flow 1: Sign in / account entry
- Flow 2: Staged Add Vehicle journey with Unknown handling, relationship, identity checks, privacy, final duplicate recheck and permanent Heritage ID + QR
- Flow 3: Garage + Vehicle Overview hub with Record, Manage, Use & Share and People destinations

Architecture rules:
- One vehicle = one permanent Heritage identity.
- QR identity is permanent; visibility changes what scanners see, not the QR.
- Exact identity matches block duplicate creation.
- Chassis identity is also guarded at database level against duplicate normalized values.
- Vehicle creation + first relationship role is moving to one atomic backend operation so partial records cannot be created.
- Provenance claims are source-labelled and disputes attach to specific claims wherever possible.
- Vehicle role and permissions remain separate concepts.
- Permanent vehicle record data is distinct from private owner workspace data.
- One visibility policy is the source of truth for QR/public/share behaviour.

This branch is a fresh implementation. The previous HGV3 pilot on main is retained only for historical reference.