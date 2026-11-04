# Beacon release review

## Pipeline audit
Gap / risk / repaired dependency: Production could run before validation completed. Added `needs: validate`, so the production review job waits for the unit test job to succeed.
Production reviewer control to configure (not demonstrated locally): Configure required reviewers in GitHub Settings > Environments > `production`; the workflow's environment name alone does not enforce approval.
Secret handling and scope: Retained the `REGISTRY_TOKEN` reference in the production job only. The review step does not use or print it. No real secret was added.

## Traceability
Why PATCH is appropriate: The contract describes v2.3.1 as a backward-compatible bug fix, so PATCH is the appropriate version component.
Candidate image and annotated Git tag: Candidate image is `beacon-service:v2.3.1`. Create annotated tag `v2.3.1` on the final code commit in Phase 3; tag has not been created yet.
Record tag -> code commit in PDF using git rev-parse commands: After tagging, record `git rev-parse v2.3.1^{}` (tag's commit) alongside `git rev-parse HEAD` at the tagged commit.
Image-tag mutability limitation: Version tags can be moved or overwritten unless the registry protects them. A digest provides stronger immutable image identity.

## Incident recovery
Evidence and rollback vs forward decision: In the supplied hypothetical fixture, 35% of requests fail across all users, errors are rising, and two of three readiness checks fail. Root cause is unknown; the prior version is verified healthy and rollback compatible with no schema migration. Roll back to restore service, then investigate the candidate. These are fixture facts, not observed telemetry.
Known-good target: `beacon-service:v2.3.0`.
Proposed kubectl command (not executed): `kubectl set image deployment/beacon beacon=beacon-service:v2.3.0`.
Recovery checks after action: Check rollout status, pod readiness and restart counts, readiness probes, and request success/error rates. Confirm service recovery and monitor for recurrence; capture evidence and investigate root cause before proposing another release.

## Release note and decision
Change scope: Three release configuration repairs only: production waits for validation, deployment identifies candidate v2.3.1, and rollback identifies known-good v2.3.0. No application change.
Local validation proof: `python -m unittest -v` passed (1 test). `python check_release.py all` passed pipeline, image, and rollback static checks. These checks do not validate GitHub settings, image availability, Kubernetes schema, or runtime health.
Open risk / owner / mitigation: Required reviewers and registry tag protection are not verified here. Repository owner: configure production environment reviewers; registry owner: protect release tags or use image digests. Confirm candidate image availability before any real release.
Current production decision and blocker: NOT APPROVED; NOT DEPLOYED. Production sign-off was not requested. Any real release remains blocked on authorized review and operational checks; this exercise performs no deployment.
Terraform vs Kubernetes responsibilities: Terraform provisions and manages infrastructure resources. Kubernetes manages application workloads such as Deployments and Pods; this exercise changes workload configuration and does not provision infrastructure.
