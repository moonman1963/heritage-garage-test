# Heritage Prototype — Flow 1–3 Technical Acceptance Matrix

Current preview target: `index-v14.html`
Backend: Supabase development branch `rebuild-v0-1`
Status: **Technical logic accepted for prototype use, subject to the explicitly deferred tests below.**

## Flow 1 — Account entry

- Sign in uses Supabase password auth.
- Sign In validates required email/password before calling Supabase and disables during the request to prevent duplicate submissions.
- Create Account validates required fields and distinguishes immediate-session success from email-verification-required state.
- Existing authenticated sessions restore Garage on reload.
- Auth-state changes return the app to Sign In if the real session is signed out or expires.
- Log out clears selected vehicle, cached overview, unsaved draft, duplicate-check state, current Flow 2 step, credentials and status messages.
- Explore Demo remains login-free and is isolated from a real authenticated session: any real session is signed out before Demo starts.
- Restoring a real session clears Demo mode before Garage loads.
- Profile is a real destination and does not incorrectly route to Garage.

## Garage handoff

- Empty Garage offers Add Vehicle.
- Existing vehicles render from the authenticated user's RLS-visible records.
- Selecting a vehicle opens Vehicle Overview.
- Add Another Vehicle always starts a clean Flow 2 draft.
- Vehicle tab refuses navigation until a vehicle has been selected.
- Returning/switching to Garage clears the selected vehicle and cached overview, preventing stale state from leaking into another record.

## Flow 2 — Add Vehicle

1. Vehicle Type — selection maps to a database-constrained canonical value.
2. Category — canonical ranges are Veteran <1905, Vintage 1905–1945, Historic 1946–1980, Classic 1981–1999, Modern 2000+, Special Interest flexible.
3. Basic Details — Make/Model/Year each require a value or explicit Unknown. Year/category mismatch is blocked here with the expected category explained; the database enforces the same rule again.
4. Relationship — Owner / Co-owner / Manager-Trustee / Family-Representative supported for the initial relationship.
5. Identification — era-sensitive wording; chassis, registration, engine and body captured where applicable.
6. Duplicate Check — cannot continue before check; exact chassis match blocks creation; registration or make/model/year signals return Possible Match; Possible Match requires explicit resolution.
7. Photos — optional first-save state; upload remains deferred.
8. Review — summary reflects the current draft.
9. Privacy & Visibility — selection persists on back/forward and maps to one canonical visibility policy.
10. Final Check & Save — a second duplicate query runs immediately before atomic create.

### Flow 2 navigation

- Back from Step 1 exits to Garage and discards that unsaved draft.
- Back/Continue preserve captured draft data across steps.
- Step value is bounded to 1–10 and cannot drift to an invalid screen number.
- Starting a new Add Vehicle flow clears previous validation/error messages.

### Duplicate safety

- Changing make/model/year/chassis/registration after a duplicate check invalidates the previous result.
- Exact Match disables Continue and routes the user toward claim/request access or dispute rather than creating a second vehicle.
- Possible Match only permits new-record continuation after `None of these`.
- Claim/request-access and hold-for-review deliberately pause new-record creation.
- The first duplicate result set is fingerprinted. If the final recheck finds a new or changed Possible Match, prior `None of these` consent is invalidated and the user is returned to Step 6.
- Final check sends newly discovered Exact Matches back to Duplicate Check.
- Database has a normalized chassis uniqueness guard as the last line of defence.
- Duplicate RPC normalizes chassis and registration formatting before comparison.
- Duplicate RPC considers registration even when chassis is supplied; registration remains a review signal rather than silently becoming permanent exact identity.
- Duplicate-check RPC is executable only by authenticated users.

### Atomic creation

- Vehicle and initial relationship are created by one RPC transaction.
- Failure of either operation rolls the transaction back.
- Initial relationship supports Owner, Co-owner, Manager/Trustee or Family/Representative for the record creator.
- `created_by` is derived from authenticated user identity server-side.
- Atomic-create RPC rejects unauthenticated calls internally and anonymous execute permission has been revoked.
- Final Save disables while duplicate recheck + creation are running, preventing double-submit.
- A failed save restores the button and keeps the draft available for correction/retry.

## Flow 3 — Vehicle home

- Vehicle Overview receives the permanent Heritage ID and QR token.
- Live records use `get_vehicle_overview_state(vehicle_id)` as the combined RLS-aware overview source.
- Completeness, Identity Confidence and Provenance Confidence are separate metrics.
- Lifecycle is displayed separately from stackable workflow flags.
- Record / Manage / Use & Share / People routes are distinct.
- Story, Provenance, Documents, Media, Maintenance, Projects, Costs/Valuation, Journeys, Market, Transfer and People destinations are wired.
- Recent Activity is visible from the Overview.
- Provenance explains claim challenge/review behaviour and shows pending-dispute state.
- Transfer explains permanent QR/identity persistence, incoming-owner state and transfer restrictions.
- QR is generated from the permanent vehicle QR token, not a placeholder pattern.
- Private vehicles do not silently create a public share link.
- Physical Badge entry is under QR & Sharing and pairs to the existing Vehicle ID.
- Detail/QR Back routes return to Vehicle only while a vehicle is selected; otherwise they fail safely to Garage.

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

### Visibility / QR / badge consistency

- Canonical visibility comes from the single Visibility Policy, not a cached vehicle value.
- Vehicle Overview, QR & Sharing, Use & Share and Share Vehicle read the same policy.
- Changing visibility changes scanner/public exposure, not the permanent QR token or Heritage Vehicle ID.
- Physical badges pair to the existing vehicle identity; replacement/additional badges do not create a new vehicle identity.

### Confidence metric consistency

- Identity Confidence is calculated from one backend rule rather than independently entered by different screens.
- Canonical identity-field weights total exactly 100: chassis/VIN 30, registration 20, year 15, engine 10, body 5, make 10, model 10.
- Source confidence uses one five-tier vocabulary: Government Confirmed 100%, Club Confirmed 85%, User Contributed 55%, System 40%, Unverified 25%.
- Direct vehicle-entry values start as User Contributed unless a canonical fact upgrades them.
- `under_review` identity facts are discounted to 65% of source score and `disputed` facts to 40%.
- Provenance Confidence derives from provenance facts plus sourced documents using the same source-tier scores and review/dispute discounts.
- Fact/document/direct-identifier changes refresh the relevant confidence metrics.
- `get_vehicle_confidence_breakdown(vehicle_id)` exposes both scores and the rule used.
- Record Completeness remains separate from confidence.

### Record Completeness consistency

- One backend formula uses five sections totalling exactly 100 points: Core Identity 45, Provenance/Evidence 20, Documents 20, Media 10, Profile Context 5.
- `get_record_completeness_breakdown(vehicle_id)` returns each component, total, maximum 100 and Low Information threshold.
- `record_completeness` recalculates when relevant vehicle fields, vehicle facts, documents or media change.
- Low Information is driven from the same score at `<35` and clears at `>=35`.
- There is no separate “missing 5% excluded” denominator.

## Backend invariants confirmed

- RLS enabled on core vehicle/role/fact/dispute/visibility structures.
- Visibility constrained to the canonical four states.
- Vehicle Type and Era Category constrained to canonical values.
- Known/Unknown Make, Model and Year consistency enforced.
- Year range constrained.
- Era Category / Year consistency enforced, with Special Interest as the flexible exception.
- Core foreign keys indexed.
- Duplicate active workflow flags prevented at database level.
- Conflicting simultaneous active facts for the same vehicle field prevented at database level.
- Identity weighting checked to total 100.
- Record Completeness weighting checked to total 100.

## Current test-data state

- Supabase development branch remains empty of real vehicle records, so schema/workflow testing has not contaminated production-like data.
- Demo mode exercises navigation and state logic without authentication or writes.

## Explicitly deferred acceptance work

- Physical iPhone UX: safe-area feel, spacing, scrolling, keyboard behaviour, touch targets and visual density.
- End-to-end authenticated create/read/reopen using a dedicated test account. The code paths and database permissions have been inspected and hardened, but the development branch currently has no dedicated test user/vehicle record to run this destructively end-to-end.
- Real photo upload/storage UI.
- Full CRUD implementations behind the current Flow 3 destination cards.
- Production-grade step-up authentication for high-risk actions.
- Club/registry external integrations and VIN decode.

## Acceptance conclusion

For the current prototype scope, Flows 1–3 are technically coherent: permanent vehicle identity, duplicate prevention, atomic first save, canonical visibility, confidence/completeness calculations, transfer/dispute status and navigation all have a single defined source of truth or explicit backend safeguard. The remaining work is either device-level UX acceptance or functionality intentionally outside this prototype milestone.