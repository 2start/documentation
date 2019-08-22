Update ShiftLeft Products
Moved debug tutorial into this article, update TOC and delete file
Do we want to include this in the "features" article?

GitHub - Chetan
JIRA - Preetam
Jenkins - Hugh

# Integrating ShiftLeft Ocular with 3rd Party Apps

You can integrate ShiftLeft Ocular with the 3rd party apps to make results available and exportable via standard JSON format. These results can then be imported into the security tools in use by your organization and for sharing data across the SDLC.

## Debugging ShiftLeft Ocular Scripts with JDB

```bash
export JAVA_OPTS='-Xdebug -Xrunjdwp:transport=dt_socket,address=8002,server=y,suspend=n'
./ocular.sh
```
The first line of output is `Listening for transport dt_socket at address: 8002`. You can now connect to remote debug with your favorite debugger, e.g. Intellij or jdb command line.
