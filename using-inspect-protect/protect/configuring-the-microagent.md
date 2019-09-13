# Configuring the ShiftLeft Protect Microagent

The ShiftLeft Protect Microagent runs out-of-the-box with default settings. Optionally you can configure the Microagent for your environment.

## Configuration Options

### Defaults

The ShiftLeft Protect for Java Microagent is configured with default settings that support core operations. (The defaults are identified in context below.) 

The ShiftLeft Protect for Java Microagent supports various configuration mechanisms, including `shiftleft.json`, Java properties, or environment variables, each overriding the values of the previous.

**Note**. Properties specified in the `shiftleft.json` file override the default. In addition, a property specified as a Java system property overrides that same property specified in the JSON file. Environment variables take precedence over both.

### JSON

You can configure the Microagent using the JSON file named `shiftleft.json` located in the working directory with the application binary. This file is generated when you run ShiftLeft Inspect.

Syntax: Must conform to the ShiftLeft format.

For example, to configure the proxy and log level using the `shiftleft.json` file:

```json
{
  "accessToken": "eyJhbGciOiJSUzUxMiIsInR5cCI6IkpXVCJ9.eyJl...",
  "sprId": "sl/0adb2179-40f6-443e-b346-e7b54e5efbfb/cli-hello-shiftleft-0.0.1.jar/.%2Fhello-shiftleft-0.0.1.jar/v/baseline",
  "slProxy": {
    "host": "10.8.8.10",
    "port": 443,
    "certificate": "*"
    },
  "log": {
    "level": "TRACE",
    "maxFiles": 5,
    "maxFileBytes": 10000000
  }
}
```

### Java Properties

Configure the Microagent using Java system properties specified as arguments to the JVM command.

Syntax: `-D${prop-name}=${prop-value}`

For example, to set the debug level for logging using Java properties:

```
java -javaagent:sl-microagent-x.y.z.jar -Dshiftleft.log.level=DEBUG -jar /hello-shiftleft-x.y.z.jar
```


### Environment Variables

Configure the Microagent using process environment variables passed to the JVM by its parent process.

Syntax:  `SHIFTLEFT_PROP_NAME="prop-value"`

For example, process environment variables passed to the JVM can be set in bash as follows:

```bash
export SHIFTLEFT_LOG_LEVEL=DEBUG 
```

## Required Parameters

If analysis has been performed as a separate step, then the `accessToken` and `sprId` parameters are required by the Microagent and must be passed using the `shiftleft.json` file that is generated during `analyze` by including it in the working directory with the app JAR file that you are deploying with the Microagent.

As an alternative to providing a `shiftleft.json` file, the `accessToken` and `sprId` can be provided via the environment variables `SHIFTLEFT_ACCESS_TOKEN` and `SHIFTLEFT_SPR_ID`, respectively.

### Access Token

The accessToken property represents the client access token that authorizes the Microagent to use ShiftLeft services.

### Security Profile for Runtime (SPR) ID

The SPR ID property identifies the [SPR](../../../policies/about-policy.md) to fetch from the proxy.

```json
{
  "accessToken": "${access-token-string}",
  "sprId": "${sprd-id}"
}
```

> **Important**. The required parameters `accessToken` and `sprId` must be passed using `shiftleft.json`. They cannot be passed using Java properties or environment variables. In addition, the `shiftleft.json` file must be located in the project directory where the application binary is located when you deploy the app with the Microagent. Alternatively, you can use the `SHIFTLEFT_CONFIG` environment variable to pass in the path to the `shiftleft.json` file.
>
> Example:
>
> `$ SHIFTLEFT_CONFIG=/home/myhome/shiftLeftFiles/shiftleft.json sl run -- <....>`

The `sprId` is a string containing three main parts: organization ID, application name, and commit hash. For example:

```
"sprId": "sl/418a892e-32fe-4d6e-b0cd-a44c24026b7a/org.springframework-my-rest-service-jar/f0e2bafa21a5790b1d70f6309189de6a1c888e16/baseline"
```

If you are using a SCM system such as Git, when you reanalyze the app after changing the code the commit hash changes, indicating a new version of the app is built. In this case you must redeploy the app with the Microagent using the updated `shiftleft.json` file. 

## Strictness

The strictness setting determines how the Microagent behaves if there is a disconnection between it and the proxy server. The ShiftLeft Protect for Java Microagent runs in two modes: non-strict (default) and strict.

In non-strict mode (default), the Microagent does not require the SPR at startup. In this case the app starts but may not be monitored by ShiftLeft, or in case it is, metrics generated during disconnection are ignored.

In strict mode (`"strict":"true"`) the Microagent requires the SPR to let the application run. If the Microagent loses connection with the proxy, the application methods calling the proxy are paused until connection is reestablished.

Parameter | Name
--- | ---
JSON | `strict`
JVM | `-Dshiftleft.strict`
Environment Variable | `SHIFTLEFT_STRICT`

### Strictness Values

- `false` (default): Proxy availability is not required. Event notifications can be lost and the application can run without instrumentation (if the proxy did not respond with the SPR at startup).
- `true`: Agent pauses the execution of the program until connection with the proxy is established and the SPR is passed to the Microagent.

The default strictness mode is false.

## ShiftLeft Proxy

The Microagent connects to a ShiftLeft Proxy server to obtain the SPR and push runtime metrics to ShiftLeft. The following options are available for microagent-proxy connections.

There is an idle timeout of approximately 40-60 minutes for the connection between the ShiftLeft Protect for Java Microagent and Proxy server. If the Microagent agent is idle for this period, the connection may be dropped. The Microagent  automatically reconnects when the app is run.

>**Important**. ShiftLeft Proxy is not to be confused with system proxy configuration.

### Proxy Host Name

Proxy host name or IP address.

Parameter | Name
--- | ---
JSON | `slProxy.host`
JVM | `-Dshiftleft.sl.proxy.host=`
Environment Variable | `SHIFTLEFT_SL_PROXY_HOST`

### Proxy Port

Proxy listening TCP port.

Parameter | Name
--- | ---
JSON | `slProxy.port`
JVM | `-Dshiftleft.sl.proxy.port`
Environment Variable | `SHIFTLEFT_SL_PROXY_PORT`

## Logging

This section describes configuration options for logging Microagent activity.

### Log Levels

Log level determines the level of detail that the Microagent logger outputs.

Parameter | Name
--- | ---
JSON | `log.level`
JVM | `-Dshiftleft.log.level`
Environment Variable | `SHIFTLEFT_LOG_LEVEL`

**Log Level Values** (from most to least logging information)
- `TRACE`: Finest level. Useful for technical debugging. Not for use in production environments.
- `DEBUG`: Detailed level. Useful for debugging. Not for use in production environments.
- `INFO`: Reasonable informative level. Returns information relevant to the user.
- `WARNING`: Warning informative level.
- `ERROR`: Show only errors.
- `QUIET`: (default) Does not create a log file. Logging is redirected to the app `stderr`.

### Log Files

Used to write logs to file system. Denotes the file name pattern for a rolling set of logs files. If not specified, Microagent logs are redirected to target application `stderr`.

Parameter | Name
--- | ---
JSON | `log.file`
JVM | `-Dshiftleft.log.file`
Environment Variable | `SHIFTLEFT_LOG_FILE`

### Rolling Log Files

Number of files to use in the rolling file set. Only used if logging to file system.

Parameter | Name
--- | ---
JSON | `log.maxFiles`
JVM | `-Dshiftleft.log.max.files`
Environment Variable | `SHIFTLEFT_LOG_MAX_FILES`

### Log File Size

Size limit per log file, in bytes. Only used if logging to file system.

Parameter | Name
--- | ---
JSON | `log.maxFileBytes`
JVM | `-Dshiftleft.log.max.file.bytes`
Environment Variable | `SHIFTLEFT_LOG_MAX_FILE_BYTES`

## Security

This section contains configuration parameters relative to the detection and blocking capabilities of the Microagent.

### Mode

Security mode that determines how the Microagent respond to attacks (malicious external payloads).

Parameter | Name
--- | ---
JSON | `sec.mode`
JVM | `-Dshiftleft.sec.mode`
Environment Variable | `SHIFTLEFT_SEC_MODE`

Values:
- `REPORT`(default): Do not alter application behavior, just report the detected attacks
- `BLOCK`: Block application execution when attacks are found, by throwing a java.lang.SecurityException, and report the attack

### XXE

Type of protection to adopt when parsing XML documents of non trusted origin.

Parameter | Name
--- | ---
JSON | `sec.xxe`
JVM | `-Dshiftleft.sec.xxe`
Environment Variable | `SHIFTLEFT_SEC_XXE`

Values:
  - `OFF`: No XXE protection (default)
  - `DTD`: Disable DTDs completely. Almost all XML entity attacks are prevented including denial of services (DOS) attacks such as Billion Laughs.
  - `EXTERNAL`: Disable only external DTDs and entities. This protects against XXE attacks but not against denial of services (DOS) attacks such as Billion Laughs.

### Collect Attack Information

Enables collecting full payloads of attack events. This might include sensitive information, which is stored encrypted. Disabled by default.

Parameter | Name
--- | ---
JSON | `sec.collect.attack.info`
JVM | `-Dshiftleft.sec.collect.attack.info`
Environment Variable | `SHIFTLEFT_SEC_COLLECT_ATTACK_INFO`

 Values:
  - `true`: Attack payloads are collected and sent to ShiftLeft's infrastructure for viewing in the [Vulnerability Dashboard Event Viewer](../../../using-inspect-protect/using-workflow/vulnerability-dashboard.md#event-details).
  - `false`: (default)

## HTTPS Proxy Configuration

The ShiftLeft Protect for Java Microagent supports the commonly used environment variable `https_proxy` for configuring a HTTPS proxy.

```bash
export https_proxy="http://[$user:$password@]$host:$port"
```
