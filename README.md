# Homelab

## Folder diagram

```txt
homelab/
├── README.md                    # architecture overview, diagrams embedded from docs/diagrams
├── .gitleaks.toml                # secret scanning config, run before every commit
├── .gitignore                    # excludes Pulumi.<real-stack>.yaml, node_modules
├── package.json                  # root — workspaces (bun/npm), shared devDependencies
├── tsconfig.base.json             # shared config, extended by each subproject
│
├── skills/
│   ├── homelab-security/SKILL.md
│   └── english-only-output/SKILL.md
│
├── network/                      # Pulumi project #1 — RouterOS
│   ├── Pulumi.yaml
│   ├── Pulumi.example.yaml       # redacted example (committed); real stack config lives outside the repo
│   ├── package.json
│   ├── tsconfig.json
│   ├── index.ts                  # composes the resources below
│   └── src/
│       ├── vlans.ts
│       ├── firewall.ts
│       ├── dhcp.ts
│       └── wireguard.ts
│
├── talos/                        # Pulumi project #2 — Talos node OS + cluster bootstrap
│   ├── Pulumi.yaml
│   ├── Pulumi.example.yaml
│   ├── package.json
│   ├── tsconfig.json
│   ├── index.ts                  # generates machine configs, applies, bootstraps
│   └── src/
│       ├── machine-config.ts     # shared config (cluster name, CNI, etc.)
│       ├── control-plane.ts
│       └── workers.ts
│   # stack outputs: kubeconfig, talosconfig (consumed by cluster/ via StackReference)
│
├── cluster/                      # Pulumi project #3 — Kubernetes
│   ├── Pulumi.yaml
│   ├── Pulumi.example.yaml
│   ├── package.json
│   ├── tsconfig.json
│   ├── index.ts
│   └── src/
│       ├── namespaces.ts
│       ├── networking.ts         # ingress/CNI etc.
│       ├── storage.ts
│       └── workloads/
│
├── diagrams/                     # not a Pulumi project — a standalone doc-generation script
│   ├── package.json
│   ├── tsconfig.json
│   ├── generate.ts               # entry point: builds Diagram objects with diagrams-js
│   └── src/
│       ├── network-topology.ts
│       └── cluster-architecture.ts
│
└── docs/
    └── diagrams/                  # generated SVGs — committed (safe: no real IPs/names)
```

## Why this structure?

- **Project separation** — Different stacks serve different purposes, reducing the blast
  radius if an apply fails. The stacks also form a defined execution order: network → Talos
  (bare-metal setup) → Kubernetes cluster.
- **Diagrams** — I use the diagram-generation tooling deliberately so diagrams can be
  regenerated from well-described, type-safe TypeScript objects within a single TypeScript
  ecosystem.
- **Bun workspaces** — The root `package.json` defines the workspaces, allowing a single
  `bun install` from the project root and a shared `tsconfig.base.json`. Each Pulumi project
  still has its own `package.json` because Pulumi requires a dedicated `Pulumi.yaml` and
  `package.json` for each project.

### Concerns to consider later

- Should I use Pulumi for Kubernetes? Argo CD or Flux might be better suited to a small
  project because they require less boilerplate.

### Trade-offs

#### Pulumi ESC for sensitive data storage

After a thorough analysis, I decided to store sensitive configuration and secrets in Pulumi
ESC. This solution is not perfect: SaaS providers tend to reduce their free tiers over time,
as described by the enshittification principle.

**Alternative:** If Pulumi ESC becomes too limiting, I can switch to a private repository and
reference it as a submodule.
