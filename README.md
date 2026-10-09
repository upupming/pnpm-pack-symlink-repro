# pnpm pack symlink reproduction

Minimal reproduction for pnpm 12.7+ including an internal workspace symlink in
an npm package tarball.

## Reproduce

```sh
corepack enable
pnpm install
pnpm reproduce
pnpm verify
```

`packages/demo/runtime` depends on the nested workspace package
`@repro/transform`. After installation, pnpm creates this symlink:

```text
packages/demo/runtime/node_modules/@repro/transform -> ../../../transform
```

The parent package publishes both `runtime` and `transform`. Starting with pnpm
12.7.0, `pnpm pack` preserves a symlink whose target is inside the package. The
resulting tarball therefore contains:

```text
package/runtime/node_modules/@repro/transform -> ../../../transform
```

The generated and extracted `pack/package` directory is committed so the
symlink can be inspected without running the reproduction.

## Version comparison

The same package produces no symlink entry with pnpm 12.6.0:

```sh
corepack pnpm@12.6.0 --config.pm-on-fail=ignore \
  --filter @repro/demo pack --pack-destination /tmp/pnpm-12.6
```

With pnpm 12.7.0 and newer, the tarball contains the symlink:

```sh
corepack pnpm@12.7.0 --config.pm-on-fail=ignore \
  --filter @repro/demo pack --pack-destination /tmp/pnpm-12.7
tar -tvf /tmp/pnpm-12.7/repro-demo-1.0.0.tgz | grep '^l'
```

This behavior was introduced in
[pnpm 12.7.0](https://github.com/pnpm/pnpm/releases/tag/v12.7.0):
`pnpm pack` now keeps symlinks pointing to files or directories included in the
package.
