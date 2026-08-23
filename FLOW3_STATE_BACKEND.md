# Flow 3 workflow-state backend

This pass moves Flow 3 status logic out of static UI assumptions and into the Supabase development backend.

## Automatic state synchronization

- Active disputes create/maintain `pending_dispute` vehicle flags.
- Resolving/withdrawing the final active dispute clears the `pending_dispute` flag.
- Pending/accepted/reversal-review ownership transfers create/maintain `transfer_pending`.
- Ending the final active transfer clears `transfer_pending`.
- Vehicles below the current completeness threshold automatically receive `low_information`; it clears when completeness recovers.
- Only one active flag of each type can exist per vehicle.

## Activity trail

Dispute and transfer status changes now create activity events so consequential workflow changes appear in the vehicle audit/activity history rather than existing only as badges.

## Flow 3 data contract

`get_vehicle_dashboard_state(vehicle_id)` is a SECURITY INVOKER RPC for authenticated users. RLS remains authoritative. It returns:

- active flags
- pending dispute count
- pending transfer summary (when visible to the caller)
- eight most recent activity events

The next front-end integration should use this contract for the Needs Attention area and Activity feed instead of deriving these states independently in the browser.
