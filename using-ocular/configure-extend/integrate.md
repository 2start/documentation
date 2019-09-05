
GitHub - Chetan 38 minutes
JIRA - Preetam
Jenkins - Suchakra

# Integrating ShiftLeft Ocular with 3rd Party Apps

You can integrate ShiftLeft Ocular with the 3rd party apps to make results available and exportable via standard JSON format. These results can then be imported into the security tools in use by your organization and for sharing data across the SDLC.

This article provides information on

* [Encapsulating Results in a Script for Your CI/CD Pipeline](#encapsulating-results-in-a-script-for-your-ci/cd--pipeline)
* [Automatically Creating an Issue in GitHub](#automatically-creating-an-issue-in-github)
* [Debugging ShiftLeft Ocular Scripts with JDB](#debugging-shiftleft-ocular-scripts-with-jdb)

## Encapsulating Results in a Script for Your CI/CD Pipeline


## Automatically Creating an Issue in GitHub

You can create an issue in GitHub, using the data resulting from your ShiftLeft Ocular investigation, to influence your code review process.

globals.createIssueInGitHub(flowString, globals.accessToken, globals.owner, "tarpit", "Time/Logic bomb detected. Fix before it detonates")



## Debugging ShiftLeft Ocular Scripts with JDB

```bash
export JAVA_OPTS='-Xdebug -Xrunjdwp:transport=dt_socket,address=8002,server=y,suspend=n'
./ocular.sh
```
The first line of output is `Listening for transport dt_socket at address: 8002`. You can now connect to remote debug with your favorite debugger, e.g. Intellij or jdb command line.
