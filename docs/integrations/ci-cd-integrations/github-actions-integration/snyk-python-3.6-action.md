# Snyk Python-3.6 Action

This page provides examples of using the Snyk GitHub Action for [Python (3.6)](https://github.com/snyk/actions/tree/master/python-3.6). For instructions on using the action and further information see [GitHub Actions integration](https://docs.snyk.io/integrations/ci-cd-integrations/github-actions-integration).

## Using the Snyk Python (3.6) Action to check for vulnerabilities

The examples that follow show how you can use a Snyk Python GitHub Action.

Snyk requires that Python download the dependencies before running or triggering the Snyk checks.&#x20;

The Python image checks and installs dependencies only if the manifest files are present in the current path, that is, the path from where the action is being triggered.

* If pip is present on the current path , and Snyk finds a `requirements.txt` file, then Snyk runs `pip install -r requirements.txt`.&#x20;
* If pipenv is present on the current path, and Snyk finds a `Pipfile` without a `Pipfile.lock`, then Snyk runs `pipenv update`.
* If `pyproject.toml` is present in the current path and Snyk does not find `poetry.lock` then Snyk runs `pip install poetry`.&#x20;

If manifest files are present under any location other root then they **must be installed** prior to running Snyk.

You can use the Snyk Python (3.6) Action to check for vulnerabilities as follows:

```yaml
name: Example workflow for Python-3.6 using Snyk
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/python-3.6@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

You can use the Snyk Python (3.6) Action to check for **only high severity vulnerabilities** as follows:

```yaml
name: Example workflow for Python-3.6 using Snyk
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/python-3.6@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
```

## Using the Snyk Python (3.6) Action to run snyk monitor

For an example of running `snyk monitor`, see [Snyk monitor example](https://docs.snyk.io/integrations/ci-cd-integrations/github-actions-integration#snyk-monitor-example) on the GitHub Actions integration page.

## Uploading Snyk scan results to GitHub Code Scanning using the Snyk Python (3.6) Action

Using `--sarif-file-output` [Snyk CLI option](https://docs.snyk.io/snyk-cli/cli-reference) and the [GitHub SARIF upload action](https://docs.github.com/en/code-security/secure-coding/uploading-a-sarif-file-to-github), you can upload Snyk scan results to GitHub Code Scanning as shown in the example that follows.

The Snyk Action fails when vulnerabilities are found. This would prevent the SARIF upload action from running. Thus you must use a [continue-on-error](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob\_idstepscontinue-on-error) option as shown in this example:

```yaml
name: Example workflow for Python-3.6 using Snyk
on: push
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/python-3.6@master
        continue-on-error: true # To make sure that SARIF upload gets called
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --sarif-file-output=snyk.sarif
      - name: Upload result to GitHub Code Scanning
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: snyk.sarif
```
