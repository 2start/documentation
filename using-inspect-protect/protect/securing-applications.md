# Securing Your Applications Using ShiftLeft Protect

ShiftLeft Protect secures applications against exploitation of vulnerabilities by leveraging "code informed" insights discovered by [ShiftLeft Inspect](../../introduction/products.md). ShiftLeft Protect creates runtime security specifications based on these identified vulnerabilities by

1. Deploying the [ShiftLeft Protect Microagent](#shiftleft-protect-microagent).
2. Using the [ShiftLeft JSON file](#shiftleft-json-file) to identify the appropriate Security Profile for Runtime (SPR). 
3. Connecting to the [ShiftLeft Proxy Server](#shiftleft-proxy-server) to download the SPR and send events and metrics to the [ShiftLeft Dashboard](../using-dashboard/vulnerability-dashboard.md).

## ShiftLeft Protect Microagent

The Microagent is deployed in-memory alongside your runtime application. The Microagent protects the residual issues in production and is customized to your application's specific shape and weaknesses. 

## The ShiftLeft JSON File

ShiftLeft Inspect generates the `shiftleft.json` file as part of the process of analyzing your application. This file includes the parameters `accessToken` and `sprId`, which are required by ShiftLeft Protect. 

If you analyze your application using ShiftLeft Inspect as a [separate step before runtime](../../inspect/analyzing-applications.md), the `shiftleft.json` file must be passed to ShiftLeft Protect. You do so by including it in the project directory in which the application binary is located when you deploy your application. Or you can [use the `SHIFTLEFT_CONFIG` environment variable](configuring-the-microagent.md) to pass to ShiftLeft Protect the path of the `shiftleft.json` file.

Note that if you are using a SCM system such as Git, when you reanalyze your application after code changes, the commit hash of the `sprId` parameters changes. This changed commit hash indicates that a new version of the application is built. Since you now have a new appliation version, you must redeploy your application with the Microagent using the updated `shiftleft.json` file. 

### `accessToken`

The `accessToken` property represents the client access token that authorizes ShiftLeft Protect to use ShiftLeft services.

### `sprId`

The `sprId` property identifies the [Security Profile for Runtime (SPR)](../../../policies/about-policy.md) for your application. 

```json
{
  "accessToken": "${access-token-string}",
  "sprId": "${sprd-id}"
}
```

The `sprId` is a string containing three main parts: organization ID, application name, and commit hash. For example

```
"sprId": "sl/418a892e-32fe-4d6e-b0cd-a44c24026b7a/org.springframework-my-rest-service-jar/f0e2bafa21a5790b1d70f6309189de6a1c888e16/baseline"
```

## The ShiftLeft Proxy Server

The Microagent connects to the ShiftLeft Proxy server to obtain the SPR and push runtime metrics to the ShiftLeft Dashboard. The following options are available for Microagent-proxy connections.

Note that rhere is an idle timeout of approximately 40-60 minutes for the connection between the Microagent and Proxy server. If the Microagent agent is idle for this period, the connection may be dropped. The Microagent automatically reconnects when the application is run.

>**Important**. The ShiftLeft Proxy Server is not to be confused with system proxy configuration.

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
