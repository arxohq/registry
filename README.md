# arxo registry

Public package registry for Law DSL. A store, not an evaluator: canonical
CLIR package bytes (§208) and their descriptors, nothing else.

## Layout

```
registry.json                          registry identity (registryId: arxo)
p/<name>/<version>.json                LockedPackage descriptor (lockfile.schema.json)
p/<name>/<version>/package.lawir.json  canonical package bytes §208 (compiled CLIR)
p/<name>/<version>/src/**              package sources: law.toml, .law modules,
                                       tests (.lawtest §267), A3 sidecars
p/<name>/<version>/source.json         exact file list of src/** with sha256 each
```

The descriptor's `contentHash` is the sha256 of `package.lawir.json`. The
bytes are canonical, so a consumer verifies a download by hashing the file
and comparing against its own `law.lock`.

Sources make every publication auditable and reproducible: read the `.law`
modules with their `@source` annotations, run the tests, re-lower and compare
against the published CLIR byte for byte. Pinned copies of external legal
texts (the package's `sources/` directory) are not republished here — their
provenance travels as hashes in `sources.law`, so an independently obtained
text can still be verified.

Guarantee boundary: this repository's CI proves integrity and immutability
(hashes, exact file lists, append-only history). The correspondence between
`src/**` and the compiled CLIR is enforced by the law-dsl monorepo gates
before anything is published — the registry stores proven results, it does
not recompute them.

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

## License

The contents of this registry — descriptors, compiled packages and published
sources — are licensed under the [Apache License 2.0](LICENSE).
