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
- Duplicate RPC considers registration even when chassis is supplied; registration remains a review signal rather than silently becoming permanent exact identity.

### Atomic creation

- Vehicle and initial relationship are created by one RPC transaction.
- Failure of either operation rolls the transaction back.
- Initial relationship policy supports Owner, Co-owner, Manager/Trustee or Family/Representative for the record creator.
- `created_by` is derived from authenticated user identity server-side.

## Flow 3 — Vehicle home

- Vehicle Overview receives the saved permanent Heritage ID and QR token.
- Completeness, Identity Confidence and Provenance Confidence are separate metrics.
- Record / Manage / Use & Share / People routes are distinct.
- Story, Provenance, Documents, Media, Maintenance, Projects, Costs/Valuation, Journeys, Market, Transfer and People destinations are wired.
- Provenance explains claim challenge/review behaviour.
- Transfer explains permanent QR/identity persistence and transfer restrictions.
- QR is generated from the permanent vehicle QR token, not a placeholder pattern.
- Private vehicles do not silently create a public share link.
- Physical Badge entry is under QR & Sharing and pairs to the existing Vehicle ID.

### Flow 3 state consistency

- `get_vehicle_dashboard_state(vehicle_id)` is the single backend source for Needs Attention, pending disputes, pending transfer and recent activity.
- `attention_count` is generated from the same active-flag collection shown to the user, preventing count/card drift.
- Attention items are priority ordered: dispute, identity review, transfer, estate review, maintenance, insurance, document expiry, low information.
- Pending-transfer state includes explicit restrictions: no second transfer and no permanent-identity changes while pending.
- Pending disputes automatically create/clear the `pending_dispute` vehicle flag.
- Ownership transfers automatically create/clear the `transfer_pending` vehicle flag.
- Low-information flag is synchronized from Record Completeness rather than independently calculated by different screens.
- Dispute and transfer workflow changes create activity events, so consequential status changes cannot exist only as badges.
- Only one active flag of a given type may exist per vehicle.
- Only one active canonical fact per vehicle/field may exist across `current`, `disputed` and `under_review`, preventing registration/year/source-tier contradictions across screens.

### Confidence metric consistency

- Identity Confidence is now calculated from one backend rule rather than being independently entered by different screens.
- Canonical identity-field weights total exactly 100: chassis/VIN 30, registration 20, year 15, engine 10, body 5, make 10, model 10.
- Source confidence is consistent with the five-tier vocabulary: Government Confirmed 100%, Club Confirmed 85%, User Contributed 55%, System 40%, Unverified 25%.
- Direct vehicle-entry values start as User Contributed unless a canonical fact upgrades them.
- `under_review` identity facts are discounted to 65% of their source score and `disputed` facts to 40%, so a disputed Government claim cannot continue to look fully verified.
- Provenance Confidence is calculated from non-identity provenance facts plus sourced documents using the same source-tier scores and review/dispute discounts.
- Fact and document changes automatically refresh the relevant confidence metrics.
- Direct identity edits automatically refresh Identity Confidence.
- `get_vehicle_confidence_breakdown(vehicle_id)` exposes both scores plus the rule used, so tooltips/details can explain why the number exists.
- Verified chassis/registration/year evidence now materially raises Identity Confidence; the earlier contradiction where several Government Confirmed identifiers could coexist with a very low score is no longer structurally possible.
- Record Completeness remains a separate measure of how much information is present and is not used as a proxy for confidence.

## Backend invariants now enforced

- RLS enabled on core vehicle/role/fact/dispute/visibility structures.
- Visibility values constrained to the canonical four states.
- Vehicle Type and Era Category constrained to canonical values.
- Known/Unknown Make, Model and Year consistency enforced.
- Year range constrained.
- Era Category / Year consistency enforced, with Special Interest as the flexible exception.
- Core foreign keys are indexed for expected growth.
- Duplicate active workflow flags are prevented at database level.
- Conflicting simultaneous active facts for the same vehicle field are prevented at database level.
- Identity weighting has been arithmetically checked to total 100.

## Current test-data state

- Supabase development branch is currently empty of user vehicle records, so schema/RLS/workflow tests are non-destructive.
- End-to-end authenticated write testing will be completed with a dedicated test account or during the later device acceptance pass; demo mode remains available without login.

## Deferred acceptance work

- Physical iPhone UX: spacing, safe-area feel, scrolling, keyboard behaviour and visual density.
- Real photo upload/storage UI.
- Full CRUD implementations behind the Flow 3 destination cards.
- Production-grade step-up authentication for high-risk actions.
- Club/registry external integrations and VIN decode.

These deferred items do not change the Flow 1–3 navigation or permanent-record identity model.