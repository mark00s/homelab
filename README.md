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
