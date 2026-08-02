# setup-vinyl

A GitHub Action that builds and installs
[Vinyl Cache](https://code.vinyl-cache.org/vinyl-cache/vinyl-cache)
from source, so your workflows can run `vinyld` (or, on older releases,
`varnishd`) and the rest of the Vinyl / Varnish toolchain.

The action checks out the requested `vinyl-cache` ref, builds it with the
standard `autogen` / `configure` / `make` / `make install` flow, runs
`ldconfig` on Linux so the shared libraries are picked up, and finally prints
the daemon version so the built revision is visible in the workflow log.

## Usage

```yaml
steps:
  - uses: varnishcache-friends/setup-vinyl@main
    with:
      version: main
      make-jobs: 4
```

## Inputs

| Name        | Description                            | Default |
| ----------- | -------------------------------------- | ------- |
| `version`   | Vinyl / Varnish Cache version to use.  | `main`  |
| `make-jobs` | Number of jobs to run simultaneously.  | `4`     |

### `version`

A release is mapped to the upstream tag automatically:

- **`vinyl-cache-x.y.z`** for 9.0+ - the `vinyl*` rebrand; installs the `vinyld` binary.
- **`varnish-x.y.z`** for older releases (8.x, 6.x, ...) - installs the `varnishd` binary.

A `major.minor` version (for example, `6.0`) resolves to the latest patch
release in that series. Anything else (`main`, a branch, the `last` tag) is
used as-is and uses PCRE2. The action supports both eras.

## Platform support

The action supports Linux and macOS runners. Windows is unsupported and the
action fails fast with a clear message.

Required build dependencies (`libedit`, `docutils`, `sphinx`, `graphviz`,
`automake`, `libtool`, ...) are installed and then purged again at the end of
the job so they don't linger on the runner image. The action selects PCRE1 for
Varnish 6.0/6.1 and PCRE2 for all other versions.

## Verify the build

The final step of the action prints the built daemon's version and revision.
It auto-detects which binary was installed: `vinyld` for 9.0 and newer,
`varnishd` for older releases:

```text
vinyld (vinyl-cache-9.0.1 revision ...)
```

Use it to confirm that the version you requested is the one that ended up on
the runner.
