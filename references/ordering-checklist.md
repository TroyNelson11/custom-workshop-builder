# FSC ordering checklist

## Repo checklist (all boxes before ordering)

- [ ] `site.yml` + `ui-config.yml` at repo root; `content/antora.yml` present
- [ ] Nav complete — every page linked from `nav.adoc`
- [ ] No upstream URLs or attributes (`site.yml` url/edit_url → this repo)
- [ ] Content-only: no console tabs (Terminal tab in `ui-config.yml` is fine)
- [ ] Workload resources labeled: `demo.redhat.com/application` (health),
      `demo.redhat.com/userinfo` (data back to AgnosticD) where relevant
- [ ] **Local podman Antora build succeeds and `./www` renders** — mandatory:
      AsciiDoc errors produce no warning at order time
- [ ] README states purpose, audience, and visibility
- [ ] License / internal-use marker matches the visibility decision
- [ ] Private repo: read-only token ready, valid until environment retirement

## Ordering (RHDP item: Field Content CI)

- [ ] Repository URL provided
- [ ] "Deploy Showroom?" checked
- [ ] Dedicated vs Shared chosen (Dedicated: per-user cluster, up to 10
      instances; Shared: users on one cluster — per-user namespaces)
- [ ] Instance count set
- [ ] Private repo: read-only token entered in the order form only
- [ ] (Approval gate) user confirms the order

## Verify

- [ ] Each user receives their own Showroom link and guide
- [ ] Guide renders: navigation, images, terminal tab
- [ ] `antora-builder` container logs clean (errors are silent elsewhere)
- [ ] Workload automation converged — FSC covers provision + core
      workloads; the rest is your automation
- [ ] Teardown behavior confirmed

## Community catalog (optional)

- [ ] Stub added under `catalog/untested/` (promote to `tested/` per
      template's `catalog/README.md`)

## Open placeholders — fill on first real run

- Exact order-form field names: TODO (item name confirmed: Field Content CI)
- Content-update propagation to an existing deployment (re-order vs
  redeploy): TODO
