# Production releases

| Target | Release | Recovery |
| --- | --- | --- |
| Cloudflare Worker | Run the repository's **Release** workflow on `main`. | Enter the published calendar version in `rollback_to` when the release includes a deployment record. Use the Cloudflare deployment history for older versions or for Workers that use a provider version ID. |
| Private container | Run **Release** on `main` to publish a signed digest. Deploy that digest through the infrastructure that owns the running service. | Verify the previous release's digest and signature, then pin it in the same deployment configuration and redeploy the service. |
| Package or installed binary | Run its separate manual publication workflow on `main`. | Pin the previous package version or reinstall a previously signed archive after verifying its checksum. |
| Terraform | Dispatch a plan and apply from `main`. | Review a corrective desired-state change and a fresh plan. |
| Source-only archive | Run its manual snapshot workflow. | Download the earlier source release and verify its checksum. |

A production workflow takes the exact commit selected at dispatch. New commits can land on `main` while a release runs. The selected commit must remain in main history. Serialize production releases and rollbacks for each target. Avoid resuming a superseded release as a new production deployment.

Use Conventional Commits, the Europe/Berlin `YYYY.M.N` calendar scheme, generated `CHANGELOG.md` entries, annotated release tags and GitHub Releases for versioned artifacts. Keep the tag on the tested release snapshot. Publish generated metadata onto current `main` without replacing source or force-pushing, where the repository tracks generated metadata on main. A published version's artifact and digest are fixed.

Build once for each target artifact. Run the applicable tests and scan source, dependencies and the produced artifact before publication. Block fixable HIGH and CRITICAL vulnerabilities and keep applicable secret and configuration checks. Retain reports and SBOMs. Sign the final OCI digest with GitHub OIDC and verify the expected workflow identity before moving public-facing tags. Registry aliases are convenience references. Infrastructure should record the verified digest it deploys.

Keep build jobs free of production credentials. The publish job verifies the sealed artifact before upload. A Worker release can record its exact native version in a `deployment.json` release asset with `schemaVersion`, `version`, `sourceSha`, `workerName`, and `workerVersionId`. Restore only a recorded version of the same production Worker, verify the result, and leave database state and external dependencies alone.

Use these repository implementations as references. Keep the repository's own integration tests and production gates.

- [App Worker release](https://github.com/DashioDevs/app.dashio.net/pull/49)
- [Container release](https://github.com/DashioDevs/domain-audit-service/pull/73)
- [Infrastructure apply](https://github.com/DashioDevs/infrastructure/pull/204)
