# powerplatform-alm

Promoting Power Platform solutions safely: a Dev to Sandbox to Production process with a validated rollback path.

A methodology write-up, not a project dump. It captures how a Power Apps + Power Automate solution should move from where it is built to where real people depend on it, without a bad deployment taking production down and with a way back if one does.

**Where this stands:** this is the target process, not a record of a past deployment. The app in [powerapps-employee-evaluations](https://github.com/michaellawrence-it/powerapps-employee-evaluations) moved across three environments by solution export, with each import validated before cutover. This playbook is the path from there to managed Sandbox and Production environments.

## Why this exists
Power Platform makes it easy to build in the same place users work, which makes it dangerously easy to "fix it in production." A model-driven app backed by Dataverse, security-role logic, and a dozen flows is real software; it deserves real change management. This process makes a promotion boring on purpose.

## The environments
```
DEV                 SANDBOX (staging)            PRODUCTION
build + iterate  →  full dress rehearsal      →  live, real users
unmanaged        →  managed solution import   →  managed solution import
```
- **Dev** is where the solution is authored, as an *unmanaged* solution.
- **Sandbox** is a production-like environment used as staging: the managed solution is imported exactly as production will receive it, with the same connection references and environment variables, and exercised end to end before anyone in production sees it.
- **Production** receives the same managed solution package, promoted only after Sandbox passes.

## The promotion process
```
1. EXPORT   the solution from Dev as a MANAGED package
             (managed = production cannot be edited ad hoc, so it stays
              reproducible and the next promotion is clean)

2. STAGE    import into Sandbox with a deployment settings file:
             . connection references bound to the target environment
             . environment variables set per environment (URLs, sender
               addresses, approver groups) so NOTHING is hard-coded

3. VALIDATE in Sandbox against a written checklist:
             . every flow runs green on representative data
             . security roles show the right users the right records
             . config/reference data imported and correct
             . no leftover Dev URLs, ids, or hard-coded values

4. BACK UP  export the CURRENT production managed solution first.
             THIS is the rollback artifact. You do not promote until you
             hold a restorable copy of what production is right now.

5. PROMOTE  import the validated managed solution into Production with the
             production deployment settings file. Activate flows.

6. SMOKE    a short post-promotion pass in production: trigger one real
             path of each critical flow, confirm the app opens for a
             non-admin user, confirm notifications fire.
```

## The rollback path
A promotion is only allowed if rollback is already in hand.

- **The artifact:** the pre-promotion export of the production managed solution (step 4). Because production runs a *managed* solution, re-importing the previous managed package cleanly reverts the app and flow definitions to the prior version.
- **The trigger:** any critical-path failure found in the step 6 smoke test that cannot be fixed forward in minutes.
- **The action:** re-import the backed-up managed solution, re-activate its flows, re-run the smoke test. Environment variables and connection references are re-applied from the known-good production settings file.
- **Data safety:** schema-only rollbacks are clean; when a change touched data shape, the rollback plan is written *before* promotion so a revert never orphans records.

See [rollback-checklist.md](rollback-checklist.md) for the one-page version.

## Principles that make it boring
- **Managed in every environment above Dev.** Reproducible packages, no drift, no "someone edited prod."
- **Nothing tenant-specific is hard-coded.** Connection references and environment variables carry every value that changes between environments, so the same package promotes anywhere with only a settings file.
- **Rollback is a precondition, not a reaction.** You hold the restore artifact before you touch production.
- **Validate in a production-like place, not in production.** Sandbox is the dress rehearsal; production is the performance.

## Stack
Power Platform (Power Apps, Power Automate, Dataverse) · managed solutions · environment variables + connection references · deployment settings files · Power Platform CLI (`pac`)

> Generic methodology. No employer, tenant, client, or environment identifiers. This is how, not where.
