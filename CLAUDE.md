# deployments

Deployments onto the homelab VMs.

- `compose/<vm>/<service>/compose.yaml` — services on Docker VMs.
- `clusters/prod/` — manifests for ArgoCD. **Phase 6, does not exist yet.**

## CURRENT PHASE: 1 (Base)

In phase 1 there are no workloads deployed. `vm-apps` and `vm-data` arrive in
phase 2. If you are asked to deploy something before that, say so and stop.

## Where each thing goes

| VM          | Zone      | IP           | Hosts                           | Phase |
|-------------|-----------|--------------|---------------------------------|-------|
| vm-edge     | edge      | 10.10.8.10   | cloudflared, NGINX, open-appsec | 1     |
| vm-apps     | workloads | 10.10.16.10  | application containers          | 2     |
| vm-data     | data      | 10.10.32.10  | Postgres, Redis                 | 2     |
| vm-platform | platform  | 10.10.4.20   | Prometheus, Grafana             | 3     |

The network is normative and lives in `../infrastructure/docs/zones.md`.

## Hard rules

- Everything written is in English: files, file names, comments, commits,
  branches and PRs.
- **Images pinned by digest.** Never `latest`, never a tag alone.
- **Never publish on `0.0.0.0`.** Bind to the IP of the matching zone.
- **The WAF lives on `vm-edge`** (open-appsec). Do not duplicate it in the
  service. When RKE2 arrives in phase 6, the WAF moves to the ingress, never
  duplicated.
- **Every data volume declares its backup** in a comment: B2 destination and
  frequency. Without that, the service is not deployed.
- **Secrets through SOPS+age.** Never in the compose file nor in a cleartext
  `.env`. The `.sops.yaml` suffix is mandatory.
- `vm-data` does not initiate outbound connections. If a data service needs to
  reach out, the design is wrong.
- Admin dashboards (Grafana, Vault UI, Proxmox) **are not published through
  `vm-edge`**. They are reached through the `vm-access` tunnel with
  Access+WARP.

## Promotion

Applications promote test→prod with **the same digest**. The image is never
rebuilt between environments.

## Before opening a PR

Validate with `docker compose config`. For changes touching ports or exposure,
run `homelab:network-reviewer`.
