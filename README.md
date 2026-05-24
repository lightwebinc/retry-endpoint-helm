# bitcoin-retry-endpoint Helm chart

Helm chart for [bitcoin-retry-endpoint](https://github.com/lightwebinc/bitcoin-retry-endpoint) — the BSV multicast retry endpoint that caches frames and retransmits them on NACK.

This repository packages templates, default values, JSON Schema validation, and CI workflows for the retry endpoint. The application source lives in [`bitcoin-retry-endpoint`](https://github.com/lightwebinc/bitcoin-retry-endpoint).

## Install

> The chart references `ghcr.io/lightwebinc/bitcoin-retry-endpoint:<appVersion>`. The image is delivered by Phase 1 of the containerization roadmap; until then `helm template` succeeds but `helm install` results in `ImagePullBackOff`.

```bash
helm install retry-node-1 oci://ghcr.io/lightwebinc/charts/bitcoin-retry-endpoint \
  --version 0.1.0 -n bitcoin-mcast --create-namespace \
  --set config.nackAddr=fd20::24 \
  --set 'nodeSelector.bitcoin-mcast/node=retry-1'
```

## Critical configuration

### `config.nackAddr` is effectively required

The retry endpoint advertises its NACK listen address via the BRC-126 beacon. Listeners filter ACK/MISS responses by source IPv6, so the advertised address must equal the address the kernel selects for outgoing replies.

Without `config.nackAddr` the binary attempts to auto-detect from the egress interface, which may resolve to a SLAAC address that listeners don't trust. The chart emits a `helm.sh/chart-warnings` pod annotation when `nackAddr` is empty.

In `multus` mode, set `config.nackAddr` to the per-pod fabric IPv6 from `networking.multus.fabricIPv6` (no CIDR mask).

### Redis (optional)

This chart **does not** bundle a Redis subchart. To use `config.cacheBackend=redis`, install Redis separately (e.g. `bitnami/redis`) and set `config.redisAddr=<host>:6379`.

## Networking modes

Same as the other charts — `multus` (default), `host`, `unicast` (reserved).

## Values reference

See [`values.yaml`](values.yaml). Every flag accepted by the binary is exposed under `.config`, including:

- Per-FrameVer cache TTLs (tx / block / subtree / anchor)
- All four rate-limit tiers (IP / chain / sequence / group)
- BRC-126 beacon: tier, preference, interval, scope, flags
- ACK/MISS response suppression
- BRC-132 subtree data caching
- OpenTelemetry OTLP push

## Helm test

```bash
helm test retry-node-1 -n bitcoin-mcast
```

## Release

Gated `release.yml` — `workflow_dispatch` with `confirm: RELEASE` and `production` Environment review.

## License

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
