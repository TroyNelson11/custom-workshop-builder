# FSC template layout

Grounded in `rhpds/field-sourced-content-template` — the official template
the FSC team ships. Clone it; do not rebuild the structure from scratch.

## Template tree

```
field-sourced-content-template/
├── site.yml              # Showroom Antora config          ┐ the root trio —
├── ui-config.yml         # Showroom UI configuration      │ required for
├── content/              # Antora component (the guide)   ┘ "Deploy Showroom?"
│   ├── antora.yml        # component + asciidoc attributes
│   └── modules/ROOT/     # nav.adoc, pages/, assets/
├── examples/
│   ├── helm/             # starter chart: plain K8s manifests + Helm
│   └── ansible/          # starter chart: Ansible-as-Job pattern
├── catalog/
│   ├── tested/           # item stubs the ApplicationSet lists in Argo
│   └── untested/         # stubs not yet promoted
├── docs/                 # developer guides (Ansible patterns, images)
└── README.md             # clone → customize → push → order Field Content CI
```

## The root trio

### site.yml — Antora config for Showroom

- `content.sources[0]`: `url: .`, `start_path: content` (same pattern as
  older workshop repos), `start_page: modules::index.adoc`.
- **UI bundle is remote:** `ui.bundle.url` points at the
  `rhpds/rhdp_showroom_theme` release zip with `snapshot: true`. You never
  bundle or host a UI — this replaces the old `content/docs-ui/` pattern.
- **Extensions pre-wired:** `@sntke/antora-mermaid-extension` (mermaid
  diagrams in pages) and `@andrew-jones/antora-tabs-extension` (tab blocks).
- `output.dir: ./www`.
- Set `site.url` / `edit_url` to YOUR repo — copied values pointing at
  upstream are the classic fork/new-repo mistake.

### ui-config.yml — the Showroom UI behavior

```yaml
type: showroom
default_width: 30          # left (guide) column width in %
persist_url_state: true    # browser refresh keeps UI state
view_switcher:             # guide/terminal view switching
  enabled: true
  default_mode: split
tabs:
  - name: Terminal         # the integrated terminal — supported
    path: /terminal
    port: 443
```

- Console tabs are **not supported** — do not add one. Give users `oc
  login` command blocks and dashboard URLs in the pages instead.
- Do not confuse the two "tabs": AsciiDoc *tab blocks* inside pages come
  from the Antora tabs extension in `site.yml`; the *Terminal tab* beside
  the guide comes from `ui-config.yml`.

### content/ — the Antora component

Same component pattern as older workshops: `antora.yml` (component name,
nav, `asciidoc.attributes`), `modules/ROOT/{nav.adoc, pages/, assets/}`.

- `nav.adoc` is the table of contents — `xref:<page>.adoc[]` entries,
  optionally grouped under `.Section Title` lines.
- Attribute injection: environment values (`guid`, `bastion_*`,
  `openshift_console_url`, `openshift_cluster_ingress_domain`, product
  names) live in `antora.yml` attributes and are referenced in pages as
  `{attribute}` — never hardcode them in pages.

## The workload half (examples/ + catalog/)

ArgoCD deploys your chart; the platform monitors health and feeds data
back to AgnosticD.

**Choose the pattern:**

| Pattern | Use when |
| --- | --- |
| `examples/helm/` | Deployment is expressible as Kubernetes manifests with Helm templating |
| `examples/ansible/` | You need wait-for-ready, secret generation, API calls, or conditional logic (playbooks run as a Job via a Helm chart) |

**Platform-integration labels** on your resources:

```yaml
metadata:
  labels:
    demo.redhat.com/application: "my-demo"   # health monitoring
    demo.redhat.com/userinfo: ""             # ConfigMap data (URLs, creds) flows back to AgnosticD
```

**Community catalog stub** (`catalog/tested/*.yaml`):

```yaml
name: example-ansible
displayName: Example (Ansible)
description: >-
  One-paragraph description.
owner: Your Team
repoURL: https://github.com/you/your-repo.git
revision: main
chartPath: path/to/chart
requires:
  virt: false
  rhoai: false
  aap: false
  clusterSize: sno
```

The stubs point at charts; they do not copy them. `untested/` is the
holding pen before promotion to `tested/`.

## Failure semantics (memorize)

- Missing root file (`site.yml`, `content/`) → **warning in the order
  details**; the order itself does not fail.
- AsciiDoc or `antora.yml` errors → **no warning at all** — check the
  `antora-builder` container logs.
- Therefore the local podman build is mandatory before ordering:
  `podman run --rm -v "$PWD:/antora:z" -w /antora --entrypoint antora quay.io/rhpds/antora:v1.3.0 site.yml`
  → inspect `./www`.

## Private repos

- Use a read-only token, valid until the environment is retired.
- Showroom users can read the repo with it; it is reused on pod restarts.
- The token goes in the order form only — never in files.

## Authoring conventions and Shared mode

- Content-only guides: pages, command blocks, images, terminals. No
  console embeds.
- Shared cluster: scope exercises to per-user namespaces (`{user}`
  attribute, userX pattern); never assume cluster-admin; avoid port and
  resource-name collisions. Dedicated relaxes this but caps at 10
  instances.
- For the older parasol-style pattern (self-built Pages, bundled UI,
  fork-and-repoint), see the `showroom-customization` skill's anatomy
  reference.
