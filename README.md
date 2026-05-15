# QuantCDN Purge

Purge content from [QuantCDN](https://www.quantcdn.io) using GitHub Actions. Supports purging by URL pattern or by cache key.

## Quick start

Drop a step into any workflow under `.github/workflows/`:

```yaml
- name: Purge cache after deploy
  uses: quantcdn/purge-action@v7
  with:
    customer: ${{ secrets.QUANT_CUSTOMER }}
    project: my-project
    token: ${{ secrets.QUANT_TOKEN }}
    url_pattern: /*
```

`QUANT_TOKEN` should be configured as a [repository secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

## Purging by URL pattern

Use `url_pattern` to purge by path. `/*` purges everything in the project.

```yaml
- uses: quantcdn/purge-action@v7
  with:
    customer: ${{ secrets.QUANT_CUSTOMER }}
    project: my-project
    token: ${{ secrets.QUANT_TOKEN }}
    url_pattern: /blog/*
```

## Purging by cache key

Use `cache_keys` to purge by one or more cache keys (space-separated). This requires that your origin emits a `Cache-Keys` header on cached responses. Keys are exact-match and are automatically scoped per project — there is no risk of colliding with another customer's cache.

```yaml
- uses: quantcdn/purge-action@v7
  with:
    customer: ${{ secrets.QUANT_CUSTOMER }}
    project: my-project
    token: ${{ secrets.QUANT_TOKEN }}
    cache_keys: env:test
```

Multiple keys in a single step:

```yaml
- uses: quantcdn/purge-action@v7
  with:
    customer: ${{ secrets.QUANT_CUSTOMER }}
    project: my-project
    token: ${{ secrets.QUANT_TOKEN }}
    cache_keys: env:test release:1.4.2
```

### Common pattern: per-environment purging

If a single QuantCDN project hosts multiple environments (e.g. `test.example.com` and `www.example.com`), have each environment's origin emit an environment-identifying cache key:

```
# Test responses
Cache-Keys: env:test

# Production responses
Cache-Keys: env:production
```

Then your post-deploy workflow can purge just the environment that was deployed:

```yaml
- name: Purge test environment cache
  if: github.ref == 'refs/heads/develop'
  uses: quantcdn/purge-action@v7
  with:
    customer: ${{ secrets.QUANT_CUSTOMER }}
    project: my-project
    token: ${{ secrets.QUANT_TOKEN }}
    cache_keys: env:test

- name: Purge production environment cache
  if: github.ref == 'refs/heads/main'
  uses: quantcdn/purge-action@v7
  with:
    customer: ${{ secrets.QUANT_CUSTOMER }}
    project: my-project
    token: ${{ secrets.QUANT_TOKEN }}
    cache_keys: env:production
```

## Soft purge

Set `soft_purge: true` to mark content as stale rather than evicting it immediately. Clients still receive cached content until the origin returns a fresh response.

```yaml
- uses: quantcdn/purge-action@v7
  with:
    customer: ${{ secrets.QUANT_CUSTOMER }}
    project: my-project
    token: ${{ secrets.QUANT_TOKEN }}
    cache_keys: env:test
    soft_purge: true
```

## Inputs

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `customer` | yes | — | Your customer account name |
| `project` | yes | — | Your project name |
| `token` | yes | — | Your API token |
| `url_pattern` | one of `url_pattern` or `cache_keys` | `""` | URL path pattern to purge (use `/*` to purge everything) |
| `cache_keys` | one of `url_pattern` or `cache_keys` | `""` | Space-separated cache keys to purge |
| `soft_purge` | no | `false` | Mark stale instead of immediate eviction |
| `endpoint` | no | `https://api.quantcdn.io/v1` | QuantCDN API endpoint |

`url_pattern` and `cache_keys` are mutually exclusive — provide exactly one per step. To do both, run two steps.

## Notes

- Purges are processed asynchronously and typically complete within a few seconds.
- Cache key matching is exact, not wildcard. `env:test` purges everything tagged `env:test`, but `env:` is not treated as a prefix.

## Further reading

- [Content API: Purge](https://docs.quantcdn.io/content-api/operations/purge)
- [Cache-Keys header](https://docs.quantcdn.io/api/get-started-content/)
- [GitHub Actions and secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
