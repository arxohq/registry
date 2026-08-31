# arxo registry

Public package registry for Law DSL. A store, not an evaluator: canonical
CLIR package bytes (§208) and their descriptors, nothing else.

## Layout

```
registry.json                          registry identity (registryId: arxo)
p/<name>/<version>.json                LockedPackage descriptor (lockfile.schema.json)
p/<name>/<version>/package.lawir.json  canonical package bytes §208
```

The descriptor's `contentHash` is the sha256 of `package.lawir.json`. The
bytes are canonical, so a consumer verifies a download by hashing the file
and comparing against its own `law.lock`.

## Rules

- **Publications are immutable.** Once a file under `p/` is added, it is
  never modified or deleted; a bad version is fixed by publishing the next
  version. This is enforced by CI (`check_registry.py`), not by convention.
- **Nothing is written by hand.** The only writer is the CI of the law-dsl
  monorepo (the `publish-registry` workflow): publishing means adding a line
  to `apps/registry/publish.toml` and passing the gates. The scaffolding of
  this repository (including `check_registry.py` itself) is a vendored copy
  from the monorepo, updated by the same exporter.

## Consuming

`registry://arxo/<name>/<version>` resolves to

```
https://raw.githubusercontent.com/arxohq/registry/main/p/<name>/<version>/package.lawir.json
```

and is verified against the `contentHash` of the descriptor next to it.
