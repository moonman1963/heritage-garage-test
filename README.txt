HERITAGE PROTOTYPE v0.1 — CLEAN REBUILD

Branch: rebuild-v0.1
Backend: Supabase development branch rebuild-v0-1

Scope:
- Flow 1: Sign in / account entry
- Flow 2: Add Vehicle with required Unknown handling, relationship, identity check, privacy, final duplicate recheck and permanent Heritage ID + QR
- Flow 3: Garage + Vehicle Overview hub with Record, Manage, Use & Share and People destinations

Architecture rules:
- One vehicle = one permanent Heritage identity.
- QR identity is permanent; visibility changes what scanners see, not the QR.
- Exact identity matches block duplicate creation.
- Provenance claims are source-labelled.
- Vehicle role and permissions remain separate concepts.
- Permanent vehicle record data is distinct from private owner workspace data.

This branch is a fresh implementation. The previous HGV3 pilot on main is retained only for historical reference.

Deployment note: preview trigger 2026-08-23.
