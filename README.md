# Pilot Protocol — App Catalog

Signed bundle releases for the Pilot Protocol app store.

## Status

**RC1: local-bundle install only.** The signed-Merkle-root catalog
protocol is RC2 territory. For RC1, apps are distributed as bundle
tarballs attached to [GitHub Releases](https://github.com/pilot-protocol/catalog/releases),
fetched manually by operators, and installed via:

```
pilotctl appstore verify <bundle-dir>
pilotctl appstore install <bundle-dir>
```

Each bundle is a directory containing:

- `manifest.json` — the app's manifest (id, version, declared grants, sha256 pin)
- `bin/<binary>` — the binary at the path specified by `manifest.binary.path`

The pilotctl `verify` subcommand runs both the binary's sha256 against
the manifest's pin AND the manifest's semantic Validate before any
install is accepted.

## Apps

| App ID | Version | Description | Install |
|--------|---------|-------------|---------|
| `io.pilot.aegis` | 0.1.4 | Runtime firewall for AI agents — blocks prompt injection, jailbreaks, homoglyphs, infra-impersonation. Local Qwen3-1.7B judge. | [aegis-v0.1.4](https://github.com/pilot-protocol/catalog/releases/tag/aegis-v0.1.4) |
| `io.pilot.cosift` | 0.1.2 | Web search, retrieval, and LLM-grounded research over a self-hosted corpus. | [cosift-v0.1.2](https://github.com/pilot-protocol/catalog/releases/tag/cosift-v0.1.2) |
| `io.pilot.slipstream` | 1.0.0 | Real-time crypto market data — leaderboard, signals, tape, opportunities. | [slipstream-v1.0.0](https://github.com/pilot-protocol/catalog/releases/tag/slipstream-v1.0.0) |
| `io.telepat.ideon-free` | 0.3.1 | Ideon free-tier data access. | [ideon-free-v0.3.1](https://github.com/pilot-protocol/catalog/releases/tag/ideon-free-v0.3.1) |

```bash
# Install any app
pilotctl appstore install <app-id>

# Or sideload from a bundle tarball
curl -L https://github.com/pilot-protocol/catalog/releases/download/<tag>/<bundle>.tar.gz | tar xz
pilotctl appstore verify <bundle-dir>/
pilotctl appstore install <bundle-dir>/
```

## Components

| Repo | Description |
|------|-------------|
| [pilot-protocol/app-store](https://github.com/pilot-protocol/app-store) | App store plugin: supervisor + broker + audit |
| [pilot-protocol/wallet](https://github.com/pilot-protocol/wallet) | Reference wallet (`io.pilot.wallet`) |
| pilot-protocol/catalog (this repo) | Bundle release distribution |

## Trust chain (RC1)

1. Bundle author pins binary sha256 in `manifest.json`
2. `pilotctl verify` checks pin + runs manifest Validate
3. `pilotctl install` re-verifies, stages atomically, atomic-renames
4. Supervisor re-verifies sha256 on every spawn
5. Failures audit-logged with both expected + actual hashes

The compile-time-embedded `EmbeddedCatalogPubkey` is the all-zeros
fail-closed placeholder in RC1 (signed catalogs cannot yet be
authenticated). Production builds for RC2+ must replace it.

## Known RC1 gaps

See [app-store/CHANGELOG.md](https://github.com/pilot-protocol/app-store/blob/main/CHANGELOG.md#known-gaps-deferred-to-rc2--scoped-out-of-rc1)
for the full list. Headlines: no catalog fetch + Merkle proof yet,
partial resource limits (Linux RLIMIT_NOFILE only), no encrypted-at-rest
identity, macOS + Linux only.
