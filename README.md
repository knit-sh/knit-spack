# knit-spack

A [Knit](https://github.com/knit-sh/knit) plugin that adds
[Spack](https://spack.io) as an environment manager. Import it to provision a
pinned, git-free Spack into a knit experiment and build reproducible software
environments for your setups.

## What it adds

Importing the plugin gives an experiment:

- **`knit bootstrap --spack [<ref>] --spack-packages [<ref>]`** — provision Spack
  (and its package recipes) into the experiment's `.knit` directory. A ref is a
  tag, branch, or commit; empty means the latest release. The exact commits are
  recorded in provenance metadata.
- **`knit_with_spack_env <spack.yaml>`** / **`@with_spack_env`** — declare that a
  setup builds a Spack environment from a `spack.yaml` manifest (a path, or fed
  on stdin).
- **`knit_with_spack_specs <spec>...`** / **`@with_spack_specs`** — the same, but
  from a bare spec list; the plugin writes the minimal `spack.yaml` for you.
- **`knit spack <args>...`** — run the knit-provisioned Spack, forwarding every
  argument verbatim.

Provisioning, update-mode re-provisioning, the freeze-on-built rule, the
`.activate.sh` re-entry lines for dependent jobs, and the manifest/lock
provenance are all handled by knit's generic environment-manager layer; the
plugin supplies only the Spack-specific verbs.

## Use it

Add one line near the top of your experiment, after `source knit.sh`:

```bash
source knit.sh
knit_import spack            # github.com/knit-sh/knit-spack
```

Then bootstrap with Spack and build a setup:

```bash
./experiment.sh bootstrap --spack

knit_register_setup "libs" build_libs "Build the experiment's libraries."
@with_spack_specs "cmake" "hdf5" "fftw"
knit_done
```

Pin a specific Spack release instead of the latest:

```bash
./experiment.sh bootstrap --spack v1.0.0 --spack-packages v1.0.0
```

## How it provisions

The plugin downloads Spack and `spack-packages` as GitHub tarballs with `curl`
and `tar` (knit's `knit_github_download`), so it needs **no git**. A ref is
resolved to an exact commit upstream before download, and a local-path
`repos.yaml` points Spack's builtin repository at the extracted `spack-packages`
tree. The requested refs and resolved commits are stored in the experiment's
metadata, so a reproducer provisions the same Spack.

## Requirements

- Knit (the released `knit.sh`).
- `curl` and `tar` on `PATH`.
- Whatever Spack itself needs to build your specs (a C/C++ compiler, etc.).

## Development

The plugin is a single `plugin.sh` at the repository root. Run the static
checker:

```bash
shellcheck plugin.sh
```

The test suite needs a `knit.sh` build and runs with
[bats](https://bats-core.readthedocs.io/):

```bash
KNIT_SH=/path/to/knit.sh bats tests/test_spack.sh tests/test_setup_spack.sh
```

`KNIT_SH` must point at a `knit.sh` that does not itself register Spack (the
released `knit.sh` is such a build); otherwise sourcing both the framework and
the plugin would register the Spack manager twice.
