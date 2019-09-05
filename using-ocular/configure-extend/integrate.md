
GitHub - Chetan 38 minutes
JIRA - Preetam
Jenkins - Suchakra

# Integrating ShiftLeft Ocular with 3rd Party Apps

You can integrate ShiftLeft Ocular with the 3rd party apps to make results available and exportable via standard JSON format. These results can then be imported into the security tools in use by your organization and for sharing data across the SDLC.

This article provides information on

* [Encapsulating Results in a Script for Your CI and CD Pipeline](#encapsulating-results-in-a-script-for-your-ci-and-cd--pipeline)
* [Automatically Creating an Issue in GitHub](#automatically-creating-an-issue-in-github)
* [Debugging ShiftLeft Ocular Scripts with JDB](#debugging-shiftleft-ocular-scripts-with-jdb)

## Encapsulating Results in a Script for Your CI and CD Pipeline

Encapsulating ShiftLeft Ocular results in a script, and appending it to your CI/CD pipeline, enables you to measure improvements in results across releases.

Deploy this command in your build system or any other system in your pipelie

```
./ocular.sh --script scripts.automate.sc --params jarFile=[<path>],outputFile=out.log
```

where `path` is the location and filename your application.

The command:

1. Executes ShiftLeft Ocular.
2. Encapsulates the results in a script.
3. Passes the script into a JAR file.
4. Outputs the results.

## Automatically Creating an Issue in GitHub

You can create an issue in GitHub, using the data resulting from your ShiftLeft Ocular investigation, to influence your code review process. Use the convenience function

globals.createIssueInGitHub(flowString, globals.accessToken, globals.owner, "<title text>, "<summary text>")
  
where `text` is the title and summary of the GitHub issue.

For example,

globals.createIssueInGitHub(flowString, globals.accessToken, globals.owner, "tarpit", "Time/Logic bomb detected. Fix before it detonates")

## Debugging ShiftLeft Ocular Scripts with JDB

```bash
export JAVA_OPTS='-Xdebug -Xrunjdwp:transport=dt_socket,address=8002,server=y,suspend=n'
./ocular.sh
```
The first line of output is `Listening for transport dt_socket at address: 8002`. You can now connect to remote debug with your favorite debugger, e.g. Intellij or jdb command line.