# Verification Tiers

Verification tier is independent of diff size. A small security or schema change
can require thorough verification.

## Tiers

| Tier | Use When | Minimum Evidence |
| --- | --- | --- |
| Light | Small docs/config edits with low blast radius. | Direct command or focused inspection plus diff review. |
| Standard | Default for contained implementation work. | Focused tests plus relevant lint/type/checks and diff review. |
| Thorough | Security, auth, schemas, migrations, data-loss risk, public APIs, broad refactors. | Full relevant suite, regression checks, acceptance review, and peer review when available. |

## No-Downgrade Paths

Do not downgrade verification when touched paths include:

- auth or credentials
- secrets
- schemas or migrations
- dependency manifests
- deployment config
- public APIs
- destructive data paths

## Evidence Rule

Fresh evidence beats confidence. A previous passing run is stale after edits that
could affect behavior.
