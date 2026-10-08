# Redthread Scan — GitHub Action

Run a [Redthread](https://redthreadsec.com) security scan on every push / PR and (optionally) fail
the build on findings. This is the **CI/CD gate** and the **webhook-based drift** path (a push re-scans
the asset) in one.

> Formerly `bl014h/millenniums-scan-action`: workflows that still use that name keep working (GitHub redirects it).

## Setup

1. In the app's **Continuous** view, open your asset and reveal its **trigger key**.
2. Add it to your repo as a secret, e.g. `REDTHREAD_SCAN_KEY`.
3. Note the **asset id** (shown next to the key).

## Usage

```yaml
name: security
on: [push]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: bl014h/redthread-scan-action@v1
        with:
          api-key: ${{ secrets.REDTHREAD_SCAN_KEY }}
          asset-id: "abcd1234"
          fail-on: high          # critical | high | medium | low | never
```

`fail-on: never` = **notify-only** (runs the scan, posts a summary, never fails the build).

## Inputs

| input | default | meaning |
|---|---|---|
| `api-key` | — | per-asset trigger key (repo secret) |
| `asset-id` | — | the registered asset to scan |
| `api-url` | `https://app.redthreadsec.com` | API base |
| `fail-on` | `high` | fail at/above this severity; `never` = notify-only |
| `timeout-seconds` | `1800` | max wait for the scan |
| `poll-seconds` | `20` | status poll interval |

## Outputs

`run-id`, `status`, `findings` — plus a job summary with per-severity counts.

## How it works

The Action calls the Redthread API with your per-asset trigger key: it starts a scan, polls until
the scan finishes, then applies the `fail-on` gate. No Docker image, no build step — a composite action
using `curl` + `jq` (both present on GitHub-hosted runners).

## Security

- The trigger key is **scoped to one asset** — it can only start and read scans of that asset, never
  touch the rest of your account. Rotate it any time from the Continuous view (the old key stops working).
- Scans run on the same quota / billing as manual scans; a quota or concurrency limit returns a non-2xx
  and fails the step with the reason.

## License

MIT — see [LICENSE](LICENSE). Redthread (Redthread Security Inc.) is the hosted scanning service this Action calls.
