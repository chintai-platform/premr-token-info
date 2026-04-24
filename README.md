# premr-token-info

Runtime configuration for the PREMR token frontend at
[`chintai-platform/app-monorepo`](https://github.com/chintai-platform/app-monorepo/tree/main/apps/premr-token-app).

This repo holds a single file — [`premr-config.json`](./premr-config.json) —
that the frontend fetches at runtime. Editing this file updates the price
and deployment addresses shown in the app **without a redeploy**. Clients
pick up the change within ~5 min, immediately on window refocus, or on
hard refresh.

## File format

```json
{
  "priceUsd": 55.19,
  "addresses": [
    { "chainId": 1, "contract": "0x6C436beE5F335e62110a8200F95A19d2096EFf40" },
    { "chainId": 11155111, "contract": "0xEb1CE00721Da4AB06F9F4cDAc7fC8F8aABFE7F7F" }
  ],
  "updatedAt": "2026-04-24T00:00:00Z"
}
```

| Field       | Required | Notes                                                                                                              |
| ----------- | -------- | ------------------------------------------------------------------------------------------------------------------ |
| `priceUsd`  | yes      | Positive finite number. USD per 1 PREMR. Applied regardless of chain.                                              |
| `addresses` | no       | `{ chainId, contract }[]` — one entry per chain the token is deployed on. The app picks the entry matching the wallet's current chain. Invalid entries are silently dropped; duplicates per chain id are deduped (first wins). |
| `updatedAt` | no       | ISO-8601 timestamp. Advisory — used only for editor audit.                                                         |

Common EVM chain ids:

| Chain              | Mainnet id | Testnet                    |
| ------------------ | ---------- | -------------------------- |
| Ethereum           | `1`        | Sepolia `11155111`         |
| Arbitrum One       | `42161`    | Arbitrum Sepolia `421614`  |
| Base               | `8453`     | Base Sepolia `84532`       |
| BNB Smart Chain    | `56`       | BSC Testnet `97`           |
| Polygon            | `137`      | Polygon Amoy `80002`       |
| Optimism           | `10`       | Optimism Sepolia `11155420`|
| Avalanche C-Chain  | `43114`    | Fuji `43113`               |

If `addresses` is empty or missing a chain, the corresponding balance rows
in the app show "PREMR not available on {network}".

## Raw URL the app reads

```
https://raw.githubusercontent.com/chintai-platform/premr-token-info/main/premr-config.json
```

Configured in the app via `VITE_PREMR_PRICE_CONFIG_URL`:
- `apps/premr-token-app/.env` (local dev)
- `infra/helm-environment/values/premr-token-app-dev.yaml` (dev cluster)

The frontend also defensively rewrites any 40-hex commit SHA in the ref
position to `main` at fetch time, so a pinned-commit URL still resolves to
the current `main`.

## Updating the price

1. Open [`premr-config.json`](./premr-config.json).
2. Click the pencil icon to edit. Change `priceUsd`, bump `updatedAt` to
   the current ISO timestamp.
3. Commit directly to `main` (or open a PR if branch protection is
   enabled).

## Rolling back

Use GitHub's **Revert** button on the bad commit, or locally:
```
git revert <sha> && git push
```

Full audit trail lives in `git log`.

## Adding PREMR on a new chain

Append a new `{ chainId, contract }` entry to `addresses`. The consuming
app already has ~14 EVM chains (Ethereum / Arbitrum / Base / BNB / Polygon
/ Optimism / Avalanche + their testnets) in its wagmi config, so no
frontend change is needed — it will pick up the new chain automatically
based on the wallet's current network.

If the chain isn't already in the app's configured network list, ping the
app team to add it (one-line change in `apps/premr-token-app/src/lib/appkit.ts`).

## Governance

Write access to this file is controlled by the repo's collaborator list
and branch-protection rules. Consider enabling "Require a pull request
before merging" on `main` if you want a review gate. No secrets, PATs, or
service accounts are involved — the app reads the public raw URL
directly.
