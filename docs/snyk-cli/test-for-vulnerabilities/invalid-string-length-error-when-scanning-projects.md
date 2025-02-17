# Invalid string length error when scanning projects

The invalid string length error can occur in the following situations:

* You scan **many projects** at once using the `--all-projects` option in combination with the  `--json` or `--json-file-output=<OUTPUT_FILE_PATH>` options.
* You scan a **large project** using the `--json` or `--json-file-output=<OUTPUT_FILE_PATH>` options.

You can use the following workarounds to avoid the invalid string length error.

* Remove the `--json` or  `--json-file-output=<OUTPUT_FILE_PATH>` option.
* If you require the JSON output, try increasing the severity threshold using the `--severity-threshold=<SEVERITY_LEVEL>` option. This is likely to reduce the number of less critical vulnerabilities reported and thus reduce the size of the JSON file output.
* If you are scanning multiple projects using the `--all-projects` option, try removing this option and scanning projects individually.

For detailed information about the CLI options, see the [CLI test command help](../commands/test.md).
