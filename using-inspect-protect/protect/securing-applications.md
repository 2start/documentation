# Securing Your Applications Using ShiftLeft Protect

ShiftLeft Protect secures applications against exploitation of vulnerabilities by leveraging "code informed" insights by creating runtime security specifications based on identified vulnerabilities discovered by [ShiftLeft Inspect](../../introduction/products.md) during analysis of your code. ShiftLeft Protect deploys a ShiftLeft Microagent in-memory alongside your application in production (or other environments such as staging, test, UAT). The Microagent is customized to your application's specific shape and weaknesses, and connects to the [ShiftLeft Proxy server](#the-shiftleft-proxy-server) to display events and metrics in the [ShiftLeft Dashboard](using-dashboard/vulnerability-dashboard.md).

## The ShiftLeft Proxy Server

The Microagent connects to the ShiftLeft Proxy server to obtain the Security Profile for Runtime (SPR) and push runtime metrics to the ShiftLeft Dashboard. The following options are available for Microagent-proxy connections.

There is an idle timeout of approximately 40-60 minutes for the connection between the Microagent and Proxy server. If the Microagent agent is idle for this period, the connection may be dropped. The Microagent automatically reconnects when the application is run.

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
