# Heritage v0.6 backend integration pass

This pass prepares Flow 3 to consume one RLS-aware backend payload instead of separately calculating status, confidence, and completeness in the browser.

## New overview RPC

`get_vehicle_overview_state(vehicle_id)` now returns:

- the RLS-visible vehicle record;
- the authoritative dashboard state (Needs Attention flags, pending disputes, transfer state, recent activity);
- the confidence breakdown (Identity + Provenance and the rule explanation);
- the Record Completeness breakdown.

The function is `STABLE`, is **not** `SECURITY DEFINER`, and is executable by authenticated users. Because its vehicle row is selected under normal RLS, it does not create a bypass around vehicle access permissions.

## Why this matters

Flow 3 can now load one consistent state object for the overview screen. This prevents the earlier class of contradictions where one card calculated a flag/count locally while another screen used different logic.

## Next front-end wiring

The next preview update should replace local placeholder calculations in `index-v05.html` with this RPC for authenticated vehicles. Demo mode remains local and login-free for visual testing.
