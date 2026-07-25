# Power Platform promotion + rollback checklist

The one-page version. Run top to bottom; do not skip step 4.

## Before promotion (in Sandbox / staging)
- [ ] Solution exported from Dev as a **managed** package
- [ ] Imported into Sandbox with the Sandbox **deployment settings file**
- [ ] Every flow runs green on representative test data
- [ ] Security roles verified: right users see right records, least privilege holds
- [ ] Config / reference data imported and correct
- [ ] Grep for leftovers: no Dev URLs, no hard-coded ids, no test addresses
- [ ] Environment variables reviewed for the Production target values
- [ ] Connection references identified for the Production target

## Rollback preparation (do NOT skip)
- [ ] **Export the current Production managed solution and store it as the rollback artifact**
- [ ] Record the current production solution version number
- [ ] Confirm the Production deployment settings file is current (env vars, connection refs, approver groups)
- [ ] If this change alters data shape: write the data rollback plan now

## Promotion (to Production)
- [ ] Import the validated managed solution with the **Production** deployment settings file
- [ ] Activate all flows
- [ ] Confirm connection references resolved and environment variables applied

## Smoke test (Production, immediately after)
- [ ] Trigger one real path of each critical flow; confirm green
- [ ] Open the app as a **non-admin** user; confirm it loads and scopes correctly
- [ ] Confirm notifications / emails actually send
- [ ] Confirm any calendar / list / document writes land where expected

## If the smoke test fails and cannot be fixed forward in minutes
- [ ] Re-import the backed-up Production managed solution (the rollback artifact)
- [ ] Re-activate its flows
- [ ] Re-apply the known-good Production deployment settings file
- [ ] Re-run the smoke test
- [ ] Record what failed so the next Sandbox pass catches it
