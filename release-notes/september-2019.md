# ShiftLeft September 2019 Updates

## ShiftLeft Ocular

**[New Tutorial on Identifying Incorrect or Zero Memory Allocation Bugs in C](..using-ocular/tutorials/c-allocation-bugs.md)** Arithmetic operations can lead to integer overflow (resulting in less memory being allocated) or the arithmetic operation computes to zero. These conditions may lead to exploitable vulnerabilities. This tutorial uses ShiftLeft Ocular to determine if such a condition exists.

## ShiftLeft Dashboard

* **[Improved UI](../using-inspect-protect/using-dashboard/app-list.md)**. The ShiftLeft Dashboard has a new and improved UI that makes it easier to find, view and understand your application's vulnerabilities as found by ShiftLeft Inspect and ShiftLeft Protect. 

* **[Filter Analysis Results](..//using-inspect-protect/using-dashboard/filter-results.md)**. Instead of scrolling through a list of results, you can quickly find vulnerabilities of interest by filtering. Filters can also be saved and reused as a build rule to automatically tag a build in your CI-CD pipeline based on ShiftLeft Inspect's results.

## ShiftLeft Inspect

* **[Tagging a Build Based on ShiftLeft Inspect Results](../using-inspect-protect/inspect/tag-build.md)**. A build in your CI-CD pipeline can now be automatically tagged based on ShiftLeft Inspect's code analysis results. 
