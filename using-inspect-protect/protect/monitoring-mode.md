# Using ShiftLeft Protect in Monitoring Mode

You can use ShiftLeft Protect in monitoring mode with your IAST tool.

## Example of ShiftLeft Protect Monitoring

ShiftLeft Protect detects exploits and payload, also blocks and sanitizes, vulnerabilities. For example, vulnerabilities and exposures in the [HelloShiftLeft sample application](../../introduction/helloshiftleft.md) can be tested with API access patterns described below and through example scripts provided in the [exploits directory](https://github.com/ShiftLeftSecurity/HelloShiftLeft/tree/master/exploits). 

### Sensitive Data Leaks to Log

| URL	  | Purpose |
| ------------- | ------------- |
| http://<span></span>localhost:8081/customers/1 | Returns JSON representation of Customer resource based on Id (1) specified in URL |
| http://<span></span>localhost:8081/customers | Returns JSON representation of all available Customer resources |
| http://<span></span>localhost:8081/patients | Returns JSON representation of all available patients in record |
| http://<span></span>localhost:8081/account/1 | Returns JSON representation of Account based on Id (1) specified |
| http://<span></span>localhost:8081/account | Returns JSON representation of all available accounts and their details |
	
All the above requests leak sensitive medical and PII data to the logging service. In addition other endpoints such as `/saveSettings`, `/search/user`, `/admin/login` etc. are also available. Along with the list above, users can explore variations of `GET`, `POST` and `PUT` requests sent to these endpoints.

### Remote Code Execution

An RCE can be triggered through the `/search/user` endpoint by sending a `GET` HTTP request as follows:

http://<span></span>localhost:8081/search/user?foo=new java.lang.ProcessBuilder({'/bin/bash','-c','echo 3vilhax0r>/tmp/hacked'}).start()

This creates a file `/tmp/hacked` with the content `3vilhax0r`.

### Arbitrary File Write

The [filewriteexploit.py](https://github.com/ShiftLeftSecurity/HelloShiftLeft/blob/master/exploits/filewriteexploit.py) script can be executed as follows to trigger the arbritary file writing through the `/saveSettings` endpoint:

```
$ python2 filewriteexploit.py http://localhost:8081/saveSettings testfile 3vilhax0r
```

This creates a file named `testfile` with `3vilhax0r` as its contents.

### Authentication Bypass

The [exploit.py](https://github.com/ShiftLeftSecurity/helloshiftleft/blob/master/exploits/JavaSerializationExploit/src/main/java/exploit.py) script allows an authentication bypass by exposing a deserialization vulnerability which allows administrator access:

```
$ python2 exploit.py
```

This returns the following sensitive data:

```
Customer;Month;Volume
Netflix;January;200,000
Palo Alto;January;200,000
### XSS
```

A reflected XSS vulnerability exists in the application and can be triggered using the **hidden** `/debug` endpoint as 

```
http://<span></span>localhost:8081/debug?customerId=1&clientId=1&firstName=a&lastName=b&dateOfBirth=123&ssn=123&socialSecurityNum=1&tin=123&phoneNumber=5432alert(1)
```

The result is an alert and returns the Customer object data.
