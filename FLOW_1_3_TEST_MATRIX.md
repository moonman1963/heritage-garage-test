# Heritage Prototype — Flow 1–3 Technical Test Matrix

Current preview target: `index-v05.html`
Backend: Supabase development branch `rebuild-v0-1`

## Flow 1 — Account entry

- Sign in uses Supabase password auth.
- Create Account handles immediate session or email verification state.
- Existing session restores Garage.
- Log out clears current vehicle and demo state.
- Explore Demo bypasses live writes for UX testing.
- Profile is a real destination and no longer incorrectly routes to Garage.

## Garage handoff

- Empty Garage offers Add Vehicle.
- Existing vehicles render from the authenticated user's RLS-visible records.
- Selecting a vehicle opens Vehicle Overview.
- Add Another Vehicle starts a clean Flow 2 draft.
- Vehicle tab refuses navigation until a vehicle has been selected.

## Flow 2 — Add Vehicle

1. Vehicle Type — selection required by database classification constraint.
2. Category — canonical ranges are Veteran <1905, Vintage 1905–1945, Historic 1946–1980, Classic 1981–1999, Modern 2000+, Special Interest flexible.
3. Basic Details — Make/Model/Year each require a value or explicit Unknown. Database enforces the same rule.
4. Relationship — Owner / Co-owner / Manager-Trustee / Family-Representative supported for first relationship.
5. Identification — era-sensitive wording; chassis, registration, engine and body captured where applicable.
6. Duplicate Check — cannot continue before check; exact chassis match blocks creation; registration or make/model/year signals return Possible Match; Possible Match requires explicit resolution.
7. Photos — optional first-save state; upload remains deferred.
8. Review — summary reflects current draft.
9. Privacy & Visibility — selection persists on back/forward and maps to one canonical visibility policy.
10. Final Check & Save — second duplicate query runs immediately before atomic create.

### Duplicate safety

- Changing make/model/year/chassis/registration after a duplicate check invalidates the previous result.
- Exact Match disables Continue.
- Possible Match only allows new-record continuation after `None of these`.
- Claim/request-access and hold-for-review deliberately pause new-record creation.
- Final check sends newly discovered exact/possible matches back to Duplicate Check.
- Database has a normalized chassis uniqueness guard as the last line of defence.
- Duplicate RPC now considers registration even when a chassis value was supplied; registration is a review signal, not silently treated as permanent exact identity.

### Atomic creation

- Vehicle and initial relationship are created by one RPC transaction.
- Failure of either operation rolls the transaction back.
- Initial relationship policy now supports Owner, Co-owner, Manager/Trustee or Family/Representative for the record creator.
- `created_by` is derived from authenticated user identity server-side.

## Flow 3 — Vehicle home

- Vehicle Overview receives the saved permanent Heritage ID and QR token.
- Completeness, Identity Confidence and Provenance Confidence are separate metrics.
- Low Information is only shown below the prototype threshold.
- Record / Manage / Use & Share / People routes are distinct.
- Story, Provenance, Documents, Media, Maintenance, Projects, Costs/Valuation, Journeys, Market, Transfer and People destinations are wired.
- Provenance explains claim challenge/review behaviour.
- Transfer explains permanent QR/identity persistence and transfer restrictions.
- QR is generated from the permanent vehicle QR token, not a placeholder pattern.
- Private vehicles do not silently create a public share link.
- Physical Badge entry is under QR & Sharing and pairs to the existing Vehicle ID.

## Backend invariants now enforced

- RLS enabled on core vehicle/role/fact/dispute/visibility structures.
- Visibility values constrained to the canonical four states.
- Vehicle Type and Era Category constrained to canonical values.
- Known/Unknown Make, Model and Year consistency enforced.
- Year range constrained.
- Era Category / Year consistency enforced, with Special Interest as the flexible exception.
- Core foreign keys are indexed for expected growth.

## Deferred acceptance work

- Physical iPhone UX: spacing, safe-area feel, scrolling, keyboard behaviour and visual density.
- Real photo upload/storage UI.
- Full CRUD implementations behind the Flow 3 destination cards.
- Production-grade step-up authentication for high-risk actions.
- Club/registry external integrations and VIN decode.

These deferred items do not change the Flow 1–3 navigation or permanent-record identity model.