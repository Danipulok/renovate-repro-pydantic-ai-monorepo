# Renovate reproduction: `pydantic-ai` packages get separate branches

`pyproject.toml` depends on two packages from https://github.com/pydantic/pydantic-ai: `pydantic-ai-slim` and `pydantic-evals`. Each `pydantic-evals` release pins `pydantic-ai-slim` to the same version (`pydantic-evals==2.69.0` requires `pydantic-ai-slim==2.69.0`). `uv.lock` holds both at `2.68.0`.

`renovate.json` extends `config:recommended` and sets `rangeStrategy: "update-lockfile"` for `pep621`.

## Current behaviour

Renovate plans one branch per package:

```
renovate/pydantic-ai-slim-2.x-lockfile
renovate/pydantic-evals-2.x-lockfile
```

The `pydantic-evals` branch cannot move alone. `uv lock --upgrade-package pydantic-evals` keeps the locked `pydantic-ai-slim`, so it changes nothing:

```
$ uv lock --dry-run --upgrade-package pydantic-evals
Using CPython 3.14.10
Resolved 25 packages in 1.38s
No lockfile changes detected
$ uv lock --dry-run --upgrade-package pydantic-evals --upgrade-package pydantic-ai-slim
Using CPython 3.14.10
Resolved 25 packages in 244ms
Update pydantic-ai-slim v2.68.0 -> v2.73.0
Update pydantic-evals v2.68.0 -> v2.73.0
Update pydantic-graph v2.68.0 -> v2.73.0
```

## Expected behaviour

One branch for both packages, as `group:monorepos` gives other monorepos. Adding this rule, which a `pydantic-ai` entry in `lib/data/monorepo.json` generates, gives one branch, `renovate/pydantic-ai-monorepo`:

```json
{
  "matchSourceUrls": ["https://github.com/pydantic/pydantic-ai"],
  "groupName": "pydantic-ai monorepo",
  "matchUpdateTypes": ["digest", "patch", "minor", "major"]
}
```

With the packages on separate branches, the `pydantic-evals` branch can end up already in the base branch, and Renovate then opens an empty PR from it on every release: https://github.com/renovatebot/renovate/issues/47716.

## Run it

```
docker run --rm --user "$(id -u):$(id -g)" -e HOME=/tmp -e LOG_LEVEL=debug -e LOG_FORMAT=json \
  -v "$PWD:/usr/src/app" -w /usr/src/app \
  renovate/renovate:44.257.9 --platform=local | grep -o '"branchName":"[^"]*"' | sort -u
```
