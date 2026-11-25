---
name: fsc-workshop-authoring
description: Author new hands-on workshop, lab, or demo content for the Red Hat demo platform using the Field Sourced Content (FSC) template (site.yml, ui-config.yml, content/ at the repo root), validate it locally, and order Field Content CI so Showroom deploys straight from the repo. Use when building a new workshop from scratch, publishing your own content via the demo platform, or adding an item to the FSC community catalog.
compatibility: Requires git and podman (local Antora preview). Network access to the git host, the demo platform console, and container registries.
metadata:
  author: trnelson
  version: "0.2"
---

# FSC Workshop Authoring

Field Sourced Content (FSC) is the path to publishing hands-on content on
the demo platform (RHDP). Your repo is the source of truth: ArgoCD deploys
your workload automation, and RHDP builds and deploys Showroom from the
same repo. No self-hosted Pages, no publish automation, and each user gets
their own Showroom link and guide.

## Standing restrictions

- Read-only tokens for private repos go in the order form only, never in
  the repository or any file. The token must stay valid until the
  environment is retired — Showroom users can read the repo with it, and
  it is reused when the pod restarts.
- Ordering a deployment consumes platform resources — explicit user
  approval before ordering.
- Confirm repo visibility (public vs private) with the user early.

## The template

`https://github.com/rhpds/field-sourced-content-template` — clone it and
start from `examples/helm/` (plain Kubernetes manifests with Helm
templating) or `examples/ansible/` (wait-for-ready, secret generation, API
calls, conditional logic — Ansible runs as a Job via a Helm chart). Its own
`content/` guide documents FSC itself. The full anatomy lives in
`references/fsc-layout.md`.

## Workflow

1. **Scaffold:** clone the template, keep the root trio (`site.yml`,
   `ui-config.yml`, `content/`), start from the example matching your
   deployment pattern, and customize.
2. **Author the instructions** in `content/` as content-only pages —
   integrated terminals are supported (wired in `ui-config.yml` tabs), but
   embedded OpenShift console tabs are not: write `oc login` command blocks
   and dashboard URLs instead. Tab blocks and mermaid diagrams are already
   configured as Antora extensions in the template's `site.yml`.
3. **Preview locally** (approval-free, read-only):
   `podman run --rm -v "$PWD:/antora:z" -w /antora --entrypoint antora quay.io/rhpds/antora:v1.3.0 site.yml`
   then inspect `./www`. **Do not skip this** — a bad layout only warns at
   order time, but AsciiDoc and antora.yml errors fail silently.
4. **Validate the repo** against `references/ordering-checklist.md`:
   root trio present, nav complete, no upstream-pointing attributes, and
   workload resources labeled for platform integration
   (`demo.redhat.com/application` for health monitoring,
   `demo.redhat.com/userinfo` to pass URLs and credentials back to
   AgnosticD).
5. **Decide Dedicated vs Shared with the user.** Dedicated: per-user
   cluster, up to 10 instances. Shared: users share one cluster — scope
   exercises to per-user namespaces.
6. **Order** (approval gate): the RHDP item is **Field Content CI** —
   provide your repository URL, check "Deploy Showroom?", pick cluster
   mode and instance count, and supply the read-only token if the repo is
   private. A missing root file warns in the order details; a broken build
   surfaces in the `antora-builder` container logs.
7. **Verify**: each user's Showroom link renders (navigation, images,
   terminal tab), and your automation converged — FSC covers cluster
   provision and core workloads; everything else is your responsibility.
   Confirm teardown behavior.
8. **Report**: repo URL, ordered configuration, guide links, how content
   updates propagate (confirm re-order vs redeploy on the first run and
   record it), and — optionally — the `catalog/` stub if the demo should
   appear in the community catalog.

## Community catalog

`catalog/tested/` in the template holds item **stubs** — metadata pointing
at a Helm chart (in `examples/` or someone else's GitHub): name,
displayName, description, owner, repoURL, revision, chartPath, and
`requires:` flags (virt, rhoai, aap, clusterSize). An ApplicationSet lists
them in Argo. To publish a demo for others: point a stub at your chart and
place it in `tested/` (or `untested/` first) per the template's
`catalog/README.md`.

## Updating this skill

Capture every new fact from real runs — exact order-form field names,
content-update propagation, additional quiet failure modes — into the
references and bump `metadata.version`.
