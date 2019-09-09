# ShiftLeft Command Line Interface (CLI) Reference

You need to [install the CLI](install-cli.md) before using. The CLI is invoked using the following syntax:

```
sl [global options] command [command options] [arguments...]
```

## `sl` Commands

Command | Description
--- | ---
`auth` | [Authenticate the CLI with your ShiftLeft account](../using-cli/authenticating.md).
`analyze [<path>]` | [Use ShiftLeft Inspect to analyze your application](../../using-inspect-protect/inspect/analyzing-applications.md).  `<path>` can be the path to a `.jar`, `.war` or `.ear` file, or it can be the path to a Java project directory. If `<path>` is not provided, then `.` is implied.
`help`, `h` | List ShiftLeft CLI commands or help for one command.
`install [dotnet-agent]` | Runs the ShiftLeft Protect for .NET Microagent installer.
`policy <command>` | Commands for managing [Custom Policies](../../policies/custom-policy.md).
`run -- <command>` | [Run the target command with ShiftLeft Protect's Microagent](../protect/protect-java/configuring-the-microagent.md).
`update [java-agent,libplugin]` | Update certain components of the ShiftLeft CLI, including the ShiftLeft Java Microagent (`sl update java-agent`).


## `sl` Global Options

Global Option | Environment Variable | Description
--- | --- | ---
`--verbose` | `SHIFTLEFT_VERBOSE=true` | Verbose mode.
`--sl-home <path>` | `SHIFTLEFT_HOME=<path>` | Path for config files and artifacts. If `$HOME` is set, defaults to `$HOME/.shiftleft`; otherwise to `/etc/shiftleft`.
`--help`, `-h` | | Display ShiftLeft CLI help text.
`--version`, `-v` | | Display the ShiftLeft CLI version.
None | `https_proxy=<proxy>` | Proxy configuration.

## `sl analyze` Options

`sl analyze` option | Environment Variable | Description
--- | --- | ---
`--cpg` | `SHIFTLEFT_CPG=true` | Submit application for analysis using CPG mode.
`--app <name>`, `-a <name>` | `SHIFTLEFT_APP=<name>` | Associate analysis with this application name. This name is used in the ShiftLeft UI.
`--wait`, `-w` | `SHIFTLEFT_WAIT=true` | Wait for analysis to finish before returning control.
`--java` | `SHIFTLEFT_LANG_JAVA=true` | Analyze Java code (implicit).
`--csharp` | `SHIFTLEFT_LANG_CSHARP=true` | Analyze C# code.
`--estimate` | | Produce an estimate of the number of statements of the analyzed code. Requires `--cpg` to be set. (Only valid for Java.)
`--force` | | Force analysis (instead of using a cached result).
`--dotnet-core` | | Use .NET Core. (Only valid for C#.)
`--dotnet-framework` | | Use .NET Framework. (Only valid for C#.)
`--shiftleft-json-file` | `SHIFTLEFT_JSON_FILE=<path>` | Path of Microagent configuration file `shiftleft.json`. Defaults to `shiftleft.json` (in the current working directory).
`--version-id <id>` | | Sets the version field of the [Security Profile for Runtime](../../profies/about-policy.md) (SPR) ID that's written to the config file.

## `sl run` Options

`sl run` option | Environment Variable | Description
--- | --- | ---
`--analyze <file.jar>` | `SHIFTLEFT_ANALYZE=<file.jar>` | Perform analysis before running the application. If analysis has previously been performed on this version of the application, then no further analysis is performed and the application is started immediately.
`--app <name>`, `-a <name>` | `SHIFTLEFT_APP=<name>` | Associate analysis with this application name. The name is used in the ShiftLeft UI. Only used when `--analyze` is passed.
`--cpg` | `SHIFTLEFT_CPG=true` | Submit application for analysis using CPG mode. Only used when `--analyze` is passed.
`--java` | `SHIFTLEFT_JAVA=true` | Identifies a Java application (implicit).
None | `SHIFTLEFT_CONFIG=<path> ` | Path to the `shiftleft.json` file. Defaults to `./shiftleft.json`.

## `sl install` Options

`sl install` option | Description
--- | ---
`--dotnet-core` | Use .NET Core.
`--dotnet-framework` | Use .NET Framework.

### `sl config-file` Options

`sl config-file` option | Environment Variable | Description
--- | --- | ---
`--app <name>`, `-a <name>` | `SHIFTLEFT_APP=<name>` | Associate analysis with this application name. This name is used in the ShiftLeft UI.
`--version-id <id>` | | Sets the version field of the SPR ID that's written to the config file.

## `sl policy` Commands
`sl policy` command | Description
--- | ---
`assignment` | Manage default Policies assigned to the organization and/or project.
`create` | Creates a custom Policy from a pre-defined template.
`pull` | Downloads a Policy registered in ShiftLeft's repository.
`push` | Uploads a Policy to ShiftLeft's repository.
`validate` | Checks if a Policy is valid syntactically and semantically.

## `sl policy assignment` Commands
`sl policy assignment` command | Arguments | Parameters | Description
--- | --- | --- | ---
`list` | | | Lists all Policy assignments for your organization or project that you have previously identified for use with ShiftLeft Inspect.
`remove` | | `[--project <project-name>]` | Removes the default Policy of your organization or (if the `project` parameter is provided) your organization's project.
`set` | `<policy-label>[:<policy-tag>]` | `[--project <project-name>]` | Specifies the default Policies used by ShiftLeft Inspect. `set` allows you to assign a default Policy used by ShiftLeft Inspect moving forward. If the optional `project` parameter is provided, the default Policy is only applied when ShiftLeft Inspect analyses the application specified by the `project-name` argument. If the `project` parameter is missing, the Policy is applied globally to all applications in your organization.

## `sl policy create` Options
`sl policy create` command Arguments | Description
--- | ---
`[default\|no-dictionary] [<path-to-policy-file>]` | Creates a custom Policy from a pre-defined template. Currently two templates are available: default and no-dictionary. The `default` template creates a Policy that imports all standard definitions as used by ShiftLeft, including the generic dictionary of sensitive-data variables. The `no-dictionary` template excludes these standard definitions. A new custom Policy is stored in the file defined by the `path-to-policy-file` argument, or printed to standard output if the argument is omitted.

## `sl policy info` Options
`sl policy info` command Arguments | Description
--- | ---
`[<policy-label>[:<policy-tag>]]` | Returns meta-information about all Policies with the name specified by `<policy-label>` argument. If the Policy name is omitted, meta-information of all Policies available to your organization is returned. The `policy-tag` element is optional. If the `policy-tag` element is omitted, by default the response includes all authorized Policies.

## `sl policy pull` Options
`sl policy pull` command Options | Arguments | Description
--- | --- | ---
`--hash <policy-hash>` | `[<path-to-policy-file>]` | Fetches the Policy, with the hash specified by  the `policy-hash` argument, from the ShiftLeft infrastructure, if authorized. If the `path-to-policy-file` argument is missing, the Policy is printed to the console. Returns the Policy as a text file or a bad request with detailed error information.
| | `<policy-label>[:<policy-tag>] [<path-to-policy-file>]` | Fetches the Policy, with the name specified by the `policy-label` argument, from the ShiftLeft infrastructure, if authorized. Use of the Policy tag is optional (for example `0.0.1` in `mypolicy:0.0.1`); by default the value is the latest Policy. If the `path-to-policy-file` argument is missing, the security policy is printed to the console. Returns the Policy as a text file or a bad request with detailed error information.

## `sl policy push` Options
`sl policy push` command Arguments | Description
--- | ---
`<policy-label>[:<policy-tag>] <path-to-policy-file>` | Uploads a custom Policy, with the name specified by the `policy-label` argument, to the ShiftLeft Policy repository. Use of the Policy tag is optional (for example `0.0.1` in `mypolicy:0.0.1`); by default the value is the latest Policy. Returns either OK or a bad request with detailed error information.

## `sl policy validate` Options
`sl policy validate` command Arguments | Description
--- | ---
`<path-to-policy-file>` | Verifies a custom Policy by sending it to the ShiftLeft infrastructure, which checks for syntactic and semantic errors. Returns either OK or a bad request with detailed error information.

## Artifacts Stored by `sl`

Artifact  | Description
--- | ---
`./shiftleft.json` | Analyzer output containing information necessary for running the ShiftLeft Microagent. Used to correctly associate the analysis with the specific application version.
`$SHIFTLEFT_HOME/config.json` | Credentials file generated on successful authentication.
`$SHIFTLEFT_HOME/libplugin-a.b.c.jar` | ShiftLeft Analyzer Plugin downloaded or updated during analysis.
`$SHIFTLEFT_HOME/libplugin-latest.jar` | Symlink to the latest downloaded ShiftLeft Analyzer Plugin.
`$SHIFTLEFT_HOME/sl-microagent-x.y.z.jar` | Downloaded ShiftLeft Microagent.
`$SHIFTLEFT_HOME/sl-microagent-latest.jar` | Symlink to the latest downloaded ShiftLeft Microagent.
