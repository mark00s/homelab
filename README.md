# Homelab

## Folder diagram

```
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
├── cluster/                      # Pulumi project #2 — Kubernetes
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

## Why?

- **Project separation** - different stacks have different purposes to reduce blast radius
in case of borked apply. Alas, multiple stacks can have well-defined, locical, order of
execution, i.e. network -> talos (baremetal setup) -> Kubernetes Cluster
- **Diagrams** - I am using DiagramJS on purpose, to regenerate generate well-described
diagrams trough single TypeScript ecosystem (types, yay!).
- **Bun workspaces** - inside root `package.json` as it works similar to npm — single
`bun install` in root of the project with one shared tsconfig.base.json. Every single
Pulumi project will have it's own `package.json` due to Pulumi requirement of dedicated
Pulumi.yaml+package.json per projekt.

### Concerns to consider later

- Should I use pulumi for Kubernetes? ArgoCD/Flux might be better suited for small project
(less boilerplate).

### Trade-offs

#### Pulumi ESC for sensitive data storage

After thorough analysis, I decided to store sensitive configs, secrets in Pulumi ESC.
It is far from perfection, as SaaSes tend to reduce free tiers across it's lifetime as per
enshitification principle.
**Alternative:** When Pulumi start biting, I can always switch from ESC to private repo,
and add reference it as a submodule.
