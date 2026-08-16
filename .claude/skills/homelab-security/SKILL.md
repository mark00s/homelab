---
name: homelab-security
description: Enforces security hygiene for this homelab infrastructure-as-code repo (RouterOS, Kubernetes, Pulumi/TypeScript) before anything is written, committed, or shown to the user. Triggers whenever writing or editing code, config, docs, or examples in this repo, drafting commit messages or PR descriptions, or running git add/commit/push — because this repo is public and describes a real, active home network. Always consult this before creating or editing any file in the repo.
---

# Homelab Security Guard

This repo is public and documents a real, active home network (MikroTik RouterOS + a Kubernetes cluster, managed with Pulumi/TypeScript). Treat every file the same way you'd treat code going into a company's public open-source repo: assume anyone, including someone with bad intent, will read it.

## Never write these into any file in this repo

- Credentials of any kind: RouterOS username/password, API tokens, WireGuard private keys, TLS private keys, Kubernetes service account tokens, kubeconfig contents, SSH private keys
- Real public/WAN IP addresses, or details that could be combined to infer them (e.g. exact ISP subscriber info tied to the CPE/router)
- Real internal IP ranges, VLAN IDs, hostnames, or firewall/NAT rules exactly as configured on the live network
- Anything that narrows down the physical location of the network beyond "Poland" (city, street, ISP account specifics, building details)

If you're about to write a value and you're not sure whether it falls into one of these categories, treat it as sensitive and stop — ask the user, or use a placeholder, rather than assuming it's probably fine.

## What to use instead

- IP examples: documentation-only ranges — `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` (RFC 5737) — or generic private ranges (`10.0.0.0/8`) used as illustrations, never the ranges actually in use on the network
- Hostnames: generic placeholders (`router.local`, `gw01.example`), never real device names
- Secrets: never inline in code or config. Use `pulumi config set --secret <key>` (Pulumi's built-in encryption) or environment variables populated outside the repo — never hardcode a "default" value that happens to be real
- Stack configs holding real values (e.g. a production `Pulumi.<stack>.yaml`) belong outside this repo entirely. If one is needed for local testing, create it under a path already covered by `.gitignore` and say so explicitly to the user — don't assume it's fine to commit "just this once"

## Pulumi stack outputs

Any `pulumi.export(...)` call whose value could be used to access something — a kubeconfig, a talosconfig, a connection string, a generated password — must wrap the value in `pulumi.secret()` before exporting, e.g. `pulumi.export("kubeconfig", pulumi.secret(kubeconfig))`. An unmarked output is stored in plaintext in the stack state even if the value itself came from an encrypted secret elsewhere in the program. This applies especially to the `talos/` project (kubeconfig, talosconfig) and to any `StackReference` output consumed by another project — when reviewing a `StackReference` read, check that the producing stack actually marked the output as secret, don't just assume the consuming code is careful with it.

## Before every commit

- Scan the diff for anything matching a private key block (`-----BEGIN...PRIVATE KEY-----`), a non-empty RouterOS password field, a kubeconfig `token:` / `client-key-data:` entry, or a base64 blob in a WireGuard `PrivateKey =` line
- If `gitleaks` (or a similar scanner) is already configured in this repo, run it before saying a commit is ready. If it isn't configured yet, propose adding it as a pre-commit hook as part of repo setup, rather than skipping the check
- Flag anything suspicious to the user instead of silently redacting and moving on — they need to know something looked sensitive, not just see it quietly disappear

## In documentation, READMEs, and diagrams

Architecture explanations are the most valuable part of this repo for a portfolio — keep them detailed. But keep the detail on *design decisions*, not exact live values: describe the VLAN strategy in general terms ("a dedicated VLAN isolates IoT traffic from management traffic") rather than the literal VLAN ID and subnet actually in production.
