# Test pip install (cross-arch) Action

This action runs install wheels using `pip` to test the installation of
a package in a particular architecture.

Here is an example demonstrating how to use it in a workflow with a matrix job:

```yaml
jobs:
  test-installation-cross-arch:
    name: Test installation (cross-arch)
    strategy:
      fail-fast: false
      matrix:
        architecture:
          - "arm64"
        ubuntu_version:
          - "20.04"
        python_version:
          - "3.11"
    runs-on: ubuntu-${{ matrix.ubuntu_version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Build wheels
        uses: frequenz-floss/gh-action-build-python@v0.x.x
      - name: Test installation (cross-arch)
        uses: frequenz-floss/gh-action-test-pip-install-cross-arch@v0.x.x
        with:
          architecture: ${{ matrix.architecture }}
          ubuntu_version: ${{ matrix.ubuntu_version }}
          python_version: ${{ matrix.python_version }}
```

> [!TIP]
> If you need to do some regular `nox` testing without cross-arch you can use
> the
> [`gh-action-test-pip-install`](https://github.com/frequenz-floss/gh-action-test-pip-install)
> action.

## Inputs

* `wheels_dir`: The directory where the wheels are located. Default: `dist`.

  All files with a `.whl` suffix will be installed.

  Due to the limitations of the `frequenz-floss/gh-action-run-python-in-qemu`
  action, the directory must not contain spaces or other special characters
  interpreted by the shell.

* `architecture`: The architecture to use, for example `arm64`. Required.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

* `python_version`: The Python version to use, for example `3.11`. Required.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

* `ubuntu_version`: The Ubuntu version to use, for example `20.04`. Required
  unless `dockerfile` is set.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

* `dockerfile`: The Dockerfile to use to build the Docker image, for example
  `docker/Dockerfile.arm64`. Optional.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

## Recommended use with matrix jobs

When using a matrix, it is recommended to create a dummy job to *merge* all the
matrix jobs, specially if you want to require all matrix jobs to pass to allow
merging a pull request. If you do this, you only need to add the dummy job as
a requirement and you don't need to update your requirements each time you
update your matrix.

```yaml
  test-installation-cross-arch-all:
    # The job name should match the `name` of the `test-pip-installation` job.
    name: Test pip installation (cross-arch)
    needs: ["test-installation-cross-arch"]
    runs-on: ubuntu-20.04
    steps:
      - name: Return true
        run: "true"
```

[qemu-action]: https://github.com/frequenz-floss/gh-action-run-python-in-qemu
