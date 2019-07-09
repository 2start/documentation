# Summary

* [Using ShiftLeft](README.md)
* Introduction
  * [About ShiftLeft](introduction/about.md)
  * [ShiftLeft Products](introduction/products.md)
  * [ShiftLeft Compliance and Support](introduction/support.md)
  * [ShiftLeft Requirements](introduction/requirements.md)
  * [The HelloShiftLeft Sample Application](introduction/helloshiftleft.md)
  * [Understanding the CPG](introduction/understanding-cpg.md)
* Using ShiftLeft Ocular
  * Getting Started
    * [About ShiftLeft Ocular](using-ocular/getting-started/about-shiftleft-ocular.md)
    * [Quick Start](using-ocular/getting-started/ocular-quick-start.md)
    * [Installing and Updating ShiftLeft Ocular](using-ocular/getting-started/installation.md)
    * [Memory Size Recommendations](using-ocular/getting-started/ocular-memory-size.md)
    * [Interactive and Non-Interactive Modes](using-ocular/getting-started/modes.md)
    * [Starting ShiftLeft Ocular](using-ocular/getting-started/starting.md)
    * [Creating a CPG](using-ocular/getting-started/create-cpg.md)
    * [Working with CPGs](using-ocular/getting-started/working-with-cpg.md)
    * [Deep-Dive: The CPG](using-ocular/getting-started/cpg-deep-dive.md)
  * Common Queries
    * [Analyzing Dependencies](using-ocular/common-queries/dependency-analysis.md)
    * [Analyzing Types and Packages](using-ocular/common-queries/types-packages-analysis.md)
    * [Analyzing Method Parameters](using-ocular/common-queries/parameters-analyze.md)
    * [Analyzing Methods](using-ocular/common-queries/methods-analyze.md)
    * [Uncovering Data Flows](using-ocular/common-queries/data-flows.md)
  * Use Cases
    * [Detecting 0-Day Vulnerabilities](using-ocular/use-cases/detect-0-day.md)
    * [Discovering HTTP Cookie Poisoning](using-ocular/use-cases/http-cookie-poisoning.md)
    * [Identifying Call Chains](using-ocular/use-cases/call-chains.md)
    * [Tracking Non Atomic Data Types](using-ocular/use-cases/tracking-non-atomic.md)
  * Tutorials
    * [Scanning for Deserialization Sinks](using-ocular/tutorials/deserialization.md)
    * [Analyzing a C Language Application](using-ocular/tutorials/c-language.md)
    * [Using ShiftLeft Ocular with MISRA C Applications](using-ocular/tutorials/misra-c.md)
    * [Java Vulnerable Lab](using-ocular/tutorials/java-vuln.md)
    * [Using ShiftLeft Ocular with JSP](using-ocular/tutorials/jsp.md)
    * [Debugging ShiftLeft Ocular Scripts with JDB](using-ocular/tutorials/debug-with-jdb.md)
  * [Ocular API](https://ocular.shiftleft.io/api/)
* Using ShiftLeft Inspect and ShiftLeft Protect
  * [Quick Start](using-inspect-protect/inspect-protect-quick-start.md)
  * [Windows Installer](using-inspect-protect/windows-installer.md)
  * [Using the ShiftLeft CLI](using-inspect-protect/using-cli/using-cli.md)
    * [Installing the CLI](using-inspect-protect/using-cli/install-cli.md)
    * [Authenticating with ShiftLeft](using-inspect-protect/using-cli/authenticating.md)
    * [CLI Reference](using-inspect-protect/using-cli/cli-reference.md)
  * Analyzing Your Code
    * [Analyzing Applications in CI](using-inspect-protect/inspect/analyzing-applications-in-ci.md)
    * [Using Custom Policies with ShiftLeft Inspect](using-inspect-protect/inspect/custom-policies.md)
  *  Protecting Your Runtime Application
     * [ShiftLeft Protect for Java](using-inspect-protect/protect-java/jvm-based-environments.md)
       * [Configuring the Microagent](using-inspect-protect/protect-java/configuring-the-microagent.md)
       * [Tomcat Integration](using-inspect-protect/protect-java/tomcat-integration.md)
       * [Websphere Configuration](using-inspect-protect/protect-java/websphere-configuration.md)
       * [Wildfly Configuration](using-inspect-protect/protect-java/wildfly-configuration.md)
  * Integrating ShiftLeft with Your CI/CD System
    * [Integrating with Jenkins Builds](using-inspect-protect/integrating-with-shiftleft/integrating-jenkins-builds/integrating-jenkins-builds.md)
      * [Configure Final Build Step](using-inspect-protect/integrating-with-shiftleft/integrating-jenkins-builds/configure-final-build-step.md)
      * [Configure Post Build Task](using-inspect-protect/integrating-with-shiftleft/integrating-jenkins-builds/configure-post-build-task.md)
      * [Verify Jenkins Integration](using-inspect-protect/integrating-with-shiftleft/integrating-jenkins-builds/verify-jenkins-integration.md)
    * [Integrating with Bamboo Builds](using-inspect-protect/integrating-with-shiftleft/integrating-bamboo-builds.md)
    * [Integrating with Docker Builds](using-inspect-protect/integrating-with-shiftleft/integrating-docker-builds.md)
    * [Integrating with GoCD Builds](using-inspect-protect/integrating-with-shiftleft/integrating-gocd-builds.md)
    * [Integrating with TeamCity Builds](using-inspect-protect/integrating-with-shiftleft/integrating-teamcity-builds.md)
    * [Integrating with Travis Builds](using-inspect-protect/integrating-with-shiftleft/integrating-travis-builds.md)
  * Application Programming Interfaces (APIs)
    * [Identifying Code Vulnerabilities using the API](using-inspect-protect/api/vulnerabilities_api.md)
  * Using ShiftLeft in Your Workflow
    * [Integrating Jira](using-inspect-protect/using-workflow/jira-integration.md)
    * [Vulnerability Dashboard](using-inspect-protect/using-workflow/vulnerability-dashboard.md)
* ShiftLeft Policies
  * [Policy Language](policies/spl.md)
* Monthly Updates
  * [June 2019](release-notes/june-2019.md)
  * [May 2019](release-notes/may-2019.md)
  * [April 2019](release-notes/april-2019.md)
  * [March 2019](release-notes/march-2019.md)
