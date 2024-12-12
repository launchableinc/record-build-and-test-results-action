# Launchable record build and test results action

[Launchable](https://www.launchableinc.com/) is a software development intelligence platform currently focused on continuous integration (CI). Using data from your CI runs, Launchable provides various features to speed up your testing workflow so you can ship high quality software faster.

This action sends test results and Git commit graph changes from your GitHub Actions workflow to your Launchable workspace so you can use Launchable's features including:

1. Test Notifications
2. Test Insights
3. Predictive Test Selection

# Usage

1. Create an API key on the Settings page of your Launchable workspace.
2. Create an [encrypted secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository) called `LAUNCHABLE_TOKEN` with your Launchable API key as its value.
3. Update your `actions/checkout` usage if needed (see below).
3. Add this action to your workflow. Add it after you run tests, like in the example below.
4. Add the `with:`, `if:`, and `env:` sections as shown below.
5. Set your `test_runner` and `report_path` values per [this doc](https://www.launchableinc.com/docs/sending-data-to-launchable/using-ci-integrations/using-the-launchable-github-action/#add-the-launchable-action).
6. Run the workflow.
7. If successful, you should see a link to the Launchable dashboard in the console output of the action.

Note: Depending on your test runner, you might need to modify your test runner command to ensure it creates test reports that Launchable accepts per [this doc](https://www.launchableinc.com/docs/sending-data-to-launchable/using-ci-integrations/using-the-launchable-github-action/#update-your-test-runner-command).

```yaml
name: Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  LAUNCHABLE_DEBUG: 1
  LAUNCHABLE_REPORT_ERROR: 1

jobs:
  tests:
    strategy:
      matrix:
        os: [ 'ubuntu-latest', 'windows-latest']
        version: ['v1', 'v2']
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Test
        run: <YOUR TEST COMMAND HERE>
      - name: Record build and test results action
        uses: launchableinc/record-build-and-test-results-action@v1.0.0
        with:
          report_path: .
          test_runner: <YOUR TEST RUNNER HERE>
          flavors: 'os=${{ matrix.os }}, version=${{ matrix.version }}'
        if: always()
        env:
          LAUNCHABLE_TOKEN: ${{ secrets.LAUNCHABLE_TOKEN }}
```

## Usage with `actions/checkout`

If you use `actions/checkout`, set fetch-depth: 0 as shown in the example below. Without this setting, Launchable will not receive information about all your commits.

```yaml
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
```

## Example

Refer to [go-test example](./.github/workflows/go-test-example.yaml) for example use of the action.

# Inputs

## Required

### `test_runner`

The name of your test runner. Our supported test runners are [here](https://www.launchableinc.com/docs/sending-data-to-launchable/using-ci-integrations/using-the-launchable-github-action/).

### `report_path`

Path to the test report(s) generated by your test runner. Refer to [here](https://www.launchableinc.com/docs/sending-data-to-launchable/using-ci-integrations/using-the-launchable-github-action/) for examples.

Note: Depending on your test runner, you might need to modify your test runner command to ensure it creates test reports that Launchable accepts per [this doc](https://www.launchableinc.com/docs/sending-data-to-launchable/using-ci-integrations/using-the-launchable-github-action/#update-your-test-runner-command).

## Optional

### `source_path`

Path to a local Git repository/workspace. Default `.`.

### `build_name`

[Build](https://docs.launchableinc.com/concepts/build) name that you can give to the current software. Default is the Git SHA revealed by GitHub Actions.

### `max_days`

The maximum number of days to collect commits retroactively. Default `30`.

### `no_submodules`

Flag to stop collecting build information from Git Submodules. Default `false`, which means the submodules are collected.

### `no_build`

Flag to record test session without recording build. Default `false`, which means the build is recorded.

### `python_version`

Python version for the Launchable CLI to use. Default is `3.10`. Change this if your workflow requires a specific Python version.

### `test_session_name`

Test session name that you can give to the value to [`--test-session-name`](https://www.launchableinc.com/docs/resources/cli-reference/#3500669-record-session) option of the `record session` command. (Optional)

### `flavors`

The value that you can give to the value to ['--flavor](https://www.launchableinc.com/docs/resources/cli-reference/#3500669-record-session) option to the `record session` command. e.g.) `os=linux, version=v1` (Optional)

### `test_suite`

The value that you can give to the value to ['--test-suite](https://www.launchableinc.com/docs/resources/cli-reference/#3500669-record-session) option to the `record session` command. e.g.) `test_suite=e2e` (Optional)

# License
Launchable data collection action is licensed under [Apache license](./LICENSE).
