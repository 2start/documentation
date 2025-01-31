# Detecting 0-Day Vulnerabilities

A 0-day vulnerability is unknown to, or unaddressed by, developers and security researchers, and is considered a severe threat. Until an 0-day vulnerability is identified and mitigated, hackers can exploit it.

This use case is based on CVE-2018-19859, a vulnerability allowing an attacker to execute arbitrary file
writes through [OpenRefine](https://github.com/OpenRefine/OpenRefine/). The process is:

1. [Create the Code Property Graph (CPG)](#creating-the-cpg)
2. [Identify sources](#identifying-sources)
3. [Look for the sink](#looking-for-the-sink)
4. [Detect a vulnerable flow](#detecting-a-vulnerable-flow)
5. [Verify the vulnerability](#verifying-the-vulnerability)

## CVE-2018-19859

CVE-2018-19859 describes a directory traversal attack that can be exploited as
an arbitrary file write. The vulnerability is rooted in an unsafe handling of ZIP
files, a [vulnerability pattern](http://phrack.org/issues/34/5.html#article)
often seen by security researchers, analysts, and penetration testers.

OpenRefine is described by its
authors as "a free, open source power tool for working with messy data and
improving it". A common use case for OpenRefine is the sanitization of messy
public data sets prior to statistical calculations, for which it provides
features such as importing and exporting of data that may be scattered among
multiple files or archives.


## Creating the CPG

OpenRefine is not distributed as a single JAR file, as expected by the ShiftLeft Ocular tool `java2cpg`.
However, `java2cpg` does not require its input file to comply with a specific 
file structure as long as its input is provided in the ZIP format. This format is basically
equivalent to a JAR archive; a ZIP file with a `.jar` suffix containing the
`.class` files of interest is sufficient. Based on the OpenRefine sources, which
you can download as a `.tar.gz` archive, prepare the `.jar` for `java2cpg`
by executing:

```
wget https://github.com/OpenRefine/OpenRefine/releases/download/3.1/openrefine-linux-3.1.tar.gz
tar xfz openrefine-linux-3.1.tar.gz
find openrefine-3.1 -name "*.class" | zip openrefine.jar -@
```

Then run `java2cpg` with the `-w` flag, followed by a comma-separated list of
package-names (`com.google.refine`, `org.openrefine`) to be included in the CPG. 
Only the parts of the application with the package prefixes `com.google.refine`
and `org.openrefine` are included in the CPG. For relatively large applications
such as OpenRefine, focusing on specific application parts can save computing and
analysis time.

```
ocular> ./java2cpg.sh openrefine.jar -w com.google.refine,org.openrefine -nb -o openrefine.bin.zip
```

Finally, load the newly created CPG into ShiftLeft Ocular

```
./ocular.sh
ocular> loadCpg("openrefine.bin.zip")
[..]
```

## Identifying Sources

Input sources (importers) represent program points where potentially malicious
(attacker-controlled) data may enter the system. Using ShiftLeft Ocular, you can search and define an input source.

OpenRefine relies on importing and exporting
data. Use the search strategy to look for imports by identifying all methods that contain the substring `Import` in their
full method name. This search strategy returns 502 methods, too large a number to be inspected manually. You can filter the search in order to further narrow down the number of methods by

```
ocular> cpg.method.fullName(".*Import.*").toList.size
res1: Int = 502
```

Based on the naming convention often found in [Java
Servlet](https://en.wikipedia.org/wiki/Servlet) code, the
list of methods is narrowed by including only methods accessible via HTTP. This is done through the
addition of `do(Get|Post)` to the search pattern: HTTP Get handlers are named
`doGet`, while HTTP POST handlers are named `doPost`. By executing this query, the number of results is reduced to only 13 methods, a small enough number
to inspect manually.

```
ocular> cpg.method.fullName(".*Import.*do(Get|Post).*").toList.size
res2: Int = 13
```

Some methods are stored in a class `DefaultImportingController`
which, as suggested by its name, may be interesting from a security standpoint.
The query is enhanced by replacing `Import` with `DefaultImportingController`.
The new search result consists of a `doGet` and a `doPost` method

```
ocular> cpg.method.fullName(".*DefaultImportingController.*do(Get|Post).*").fullName.p

com.google.refine.importing.DefaultImportingController.doGet:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
com.google.refine.importing.DefaultImportingController.doPost:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
```

Based on these results, by only issuing three queries, a source is found as the starting point for the 
security analysis. This following query defines a
source by applying the search filter. To be more specific, the query also
specifies the required type of the parameters (which is `HttpServletRequest`)

```
ocular> val source = cpg.method.fullName(".*DefaultImportingController.*do(Get|Post).*").parameter.evalType(".*HttpServletRequest.*")
```

## Looking for the Sink

Sinks are security-sensitive program points to which malicious,
attacker-controlled input (coming from the sink) may flow. 

Based on the OpenRefine CVE, you can find and
define sinks with ShiftLeft Ocular, specifically 
unzipping-vulnerabilities.

Using a very basic query, first identify methods which are part of the
`zip` package; among them calls to methods with `getName` as
part of their name. As a result, the method `explodeArchive`

```
ocular> cpg.method.fullName(".*zip.*getName.*").caller.fullName.p

com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)
```

Judging from its name, the function `explodeArchive` appears to be worth
investigating. This query tags the parameter of the method 
`explodeArchive` as a sink

```
ocular> val sink = cpg.method.name("explodeArchive").parameter
```

To find a possible data flow between sources and sinks, issue a
`reachableBy` query, as in the snippet

```
ocular> sink.reachableBy(source).flows.p
```

### Resulting Flow

By issuing the `reachableBy` query, a detailed picture about
the dataflow is provided, which starts from the
`doPost` method and the parameter named `request` of type
`HttpServletRequest`. Reviewing the flow, it is apparent that OpenRefine:

1. Retrieves content from a POST request (`retrieveContentFromPostRequest`)
2. A method named `download` uses a variable named `urlString`
3. `saveStream` is consuming a `url`
4. Files are allocated (`allocateFile`)
5. The methods `tryOpenAsArchive` are called 
6. Unpacks `explodeArchive` with a `ZipInputStream` variable named `archiveIS`.

In summary, OpenRefine downloads data based on a URL, reads it as
`ZipInputStream` and tries to unpack it with the method `explodeArchive`.

```
 ________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
 | param       | type                                  | method                        | signature                                                                                                                                                                                                                                                                       |
 |=======================================================================================================================================================================================================================================================================================================================================================================|
 | request(1)  | javax.servlet.http.HttpServletRequest | doPost                        | com.google.refine.importing.DefaultImportingController.doPost:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)                                                                                                                                |
 | request     | javax.servlet.http.HttpServletRequest | doPost                        | com.google.refine.importing.DefaultImportingController.doPost:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)                                                                                                                                |
 | request(1)  | javax.servlet.http.HttpServletRequest | doLoadRawData                 | com.google.refine.importing.DefaultImportingController.doLoadRawData:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse,java.util.Properties)                                                                                                    |
 | request     | javax.servlet.http.HttpServletRequest | doLoadRawData                 | com.google.refine.importing.DefaultImportingController.doLoadRawData:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse,java.util.Properties)                                                                                                    |
 | request(1)  | javax.servlet.http.HttpServletRequest | loadDataAndPrepareJob         | com.google.refine.importing.ImportingUtilities.loadDataAndPrepareJob:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse,java.util.Properties,com.google.refine.importing.ImportingJob,org.json.JSONObject)                                       |
 | request     | javax.servlet.http.HttpServletRequest | loadDataAndPrepareJob         | com.google.refine.importing.ImportingUtilities.loadDataAndPrepareJob:void(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse,java.util.Properties,com.google.refine.importing.ImportingJob,org.json.JSONObject)                                       |
 | request(1)  | javax.servlet.http.HttpServletRequest | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | request     | javax.servlet.http.HttpServletRequest | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | param0(1)   | javax.servlet.http.HttpServletRequest | parseRequest                  | org.apache.commons.fileupload.servlet.ServletFileUpload.parseRequest:java.util.List(javax.servlet.http.HttpServletRequest)                                                                                                                                                      |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | tempFiles   | java.util.List                        | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | tempFiles   | java.util.List                        | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | this(0)     | java.util.List                        | iterator                      | java.util.List.iterator:java.util.Iterator()                                                                                                                                                                                                                                    |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | l12_0       | java.util.Iterator                    | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | l12_0       | java.util.Iterator                    | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | this(0)     | java.util.Iterator                    | next                          | java.util.Iterator.next:java.lang.Object()                                                                                                                                                                                                                                      |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | $r2         | java.lang.Object                      | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | $r2         | java.lang.Object                      | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | param1(2)   | ANY                                   | <operator>.cast               | <operator>.cast                                                                                                                                                                                                                                                                 |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | fileItem    | org.apache.commons.fileupload.FileItem| retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | fileItem    | org.apache.commons.fileupload.FileItem| retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | this(0)     | org.apache.commons.fileupload.FileItem| getInputStream                | org.apache.commons.fileupload.FileItem.getInputStream:java.io.InputStream()                                                                                                                                                                                                     |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | stream      | java.io.InputStream                   | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | stream      | java.io.InputStream                   | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | param0(1)   | java.io.InputStream                   | asString                      | org.apache.commons.fileupload.util.Streams.asString:java.lang.String(java.io.InputStream)                                                                                                                                                                                       |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | urlString   | java.lang.String                      | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | urlString   | java.lang.String                      | retrieveContentFromPostRequest| com.google.refine.importing.ImportingUtilities.retrieveContentFromPostRequest:void(javax.servlet.http.HttpServletRequest,java.util.Properties,java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress)                                         |
 | urlString(6)| java.lang.String                      | download                      | com.google.refine.importing.ImportingUtilities.download:void(java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$SavingUpdate,java.lang.String)                          |
 | urlString   | java.lang.String                      | download                      | com.google.refine.importing.ImportingUtilities.download:void(java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$SavingUpdate,java.lang.String)                          |
 | urlString(6)| java.lang.String                      | download                      | com.google.refine.importing.ImportingUtilities.download:void(java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$SavingUpdate,java.lang.String,java.lang.String)         |
 | urlString   | java.lang.String                      | download                      | com.google.refine.importing.ImportingUtilities.download:void(java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$SavingUpdate,java.lang.String,java.lang.String)         |
 | param0(1)   | java.lang.String                      | <init>                        | java.net.URL.<init>:void(java.lang.String)                                                                                                                                                                                                                                      |
 | url         | java.net.URL                          | download                      | com.google.refine.importing.ImportingUtilities.download:void(java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$SavingUpdate,java.lang.String,java.lang.String)         |
 | url         | java.net.URL                          | download                      | com.google.refine.importing.ImportingUtilities.download:void(java.io.File,org.json.JSONObject,com.google.refine.importing.ImportingUtilities$Progress,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$SavingUpdate,java.lang.String,java.lang.String)         |
 | url(2)      | java.net.URL                          | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | url         | java.net.URL                          | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | this(0)     | java.net.URL                          | getPath                       | java.net.URL.getPath:java.lang.String()                                                                                                                                                                                                                                         |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | localname   | java.lang.String                      | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | localname   | java.lang.String                      | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | param0(1)   | java.lang.String                      | append                        | java.lang.StringBuilder.append:java.lang.StringBuilder(java.lang.String)                                                                                                                                                                                                        |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | $r1         | java.lang.StringBuilder               | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | $r1         | java.lang.StringBuilder               | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | this(0)     | java.lang.StringBuilder               | append                        | java.lang.StringBuilder.append:java.lang.StringBuilder(java.lang.String)                                                                                                                                                                                                        |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | $r2         | java.lang.StringBuilder               | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | $r2         | java.lang.StringBuilder               | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | this(0)     | java.lang.StringBuilder               | toString                      | java.lang.StringBuilder.toString:java.lang.String()                                                                                                                                                                                                                             |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | localname   | java.lang.String                      | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | localname   | java.lang.String                      | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | name(2)     | java.lang.String                      | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | name        | java.lang.String                      | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | this(0)     | java.lang.String                      | substring                     | java.lang.String.substring:java.lang.String(int,int)                                                                                                                                                                                                                            |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | name        | java.lang.String                      | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | name        | java.lang.String                      | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | name        | java.lang.String                      | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | name        | java.lang.String                      | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | param1(2)   | java.lang.String                      | <init>                        | java.io.File.<init>:void(java.io.File,java.lang.String)                                                                                                                                                                                                                         |
 | file        | java.io.File                          | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | file        | java.io.File                          | allocateFile                  | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                                                                                                         |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | file        | java.io.File                          | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | file        | java.io.File                          | saveStream                    | com.google.refine.importing.ImportingUtilities.saveStream:boolean(java.io.InputStream,java.net.URL,java.io.File,com.google.refine.importing.ImportingUtilities$Progress,com.google.refine.importing.ImportingUtilities$SavingUpdate,org.json.JSONObject,org.json.JSONArray,long)|
 | file(2)     | java.io.File                          | postProcessRetrievedFile      | com.google.refine.importing.ImportingUtilities.postProcessRetrievedFile:boolean(java.io.File,java.io.File,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)                                                                       |
 | file        | java.io.File                          | postProcessRetrievedFile      | com.google.refine.importing.ImportingUtilities.postProcessRetrievedFile:boolean(java.io.File,java.io.File,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)                                                                       |
 | file(1)     | java.io.File                          | tryOpenAsArchive              | com.google.refine.importing.ImportingUtilities.tryOpenAsArchive:java.io.InputStream(java.io.File,java.lang.String,java.lang.String)                                                                                                                                             |
 | file        | java.io.File                          | tryOpenAsArchive              | com.google.refine.importing.ImportingUtilities.tryOpenAsArchive:java.io.InputStream(java.io.File,java.lang.String,java.lang.String)                                                                                                                                             |
 | param0(1)   | java.io.File                          | <init>                        | java.io.FileInputStream.<init>:void(java.io.File)                                                                                                                                                                                                                               |
 | $r7         | java.io.FileInputStream               | tryOpenAsArchive              | com.google.refine.importing.ImportingUtilities.tryOpenAsArchive:java.io.InputStream(java.io.File,java.lang.String,java.lang.String)                                                                                                                                             |
 | $r7         | java.io.FileInputStream               | tryOpenAsArchive              | com.google.refine.importing.ImportingUtilities.tryOpenAsArchive:java.io.InputStream(java.io.File,java.lang.String,java.lang.String)                                                                                                                                             |
 | param0(1)   | java.io.InputStream                   | <init>                        | java.util.zip.ZipInputStream.<init>:void(java.io.InputStream)                                                                                                                                                                                                                   |
 | $r6         | java.util.zip.ZipInputStream          | tryOpenAsArchive              | com.google.refine.importing.ImportingUtilities.tryOpenAsArchive:java.io.InputStream(java.io.File,java.lang.String,java.lang.String)                                                                                                                                             |
 | $r6         | java.util.zip.ZipInputStream          | tryOpenAsArchive              | com.google.refine.importing.ImportingUtilities.tryOpenAsArchive:java.io.InputStream(java.io.File,java.lang.String,java.lang.String)                                                                                                                                             |
 | param1(2)   | ANY                                   | <operator>.assignment         | <operator>.assignment                                                                                                                                                                                                                                                           |
 | archiveIS   | java.io.InputStream                   | postProcessRetrievedFile      | com.google.refine.importing.ImportingUtilities.postProcessRetrievedFile:boolean(java.io.File,java.io.File,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)                                                                       |
 | archiveIS   | java.io.InputStream                   | postProcessRetrievedFile      | com.google.refine.importing.ImportingUtilities.postProcessRetrievedFile:boolean(java.io.File,java.io.File,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)                                                                       |
 | archiveIS(2)| java.io.InputStream                   | explodeArchive                | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)                                                                          |
 ```

## Detecting a Vulnerable Flow

After looking for sources and sinks, you can refine your search to
detect an actual vulnerability.

A `ZipInputSteam` is already controlled, but for the vulnerability you need to find a
flow to a `File` or better a `FileOutputStream`. In this query, use the
`FileOutputStream` constructor as a sink, more specifically, as the first parameter of the
constructor.


```
val source = cpg.method.name("explodeArchive").parameter
val sink = cpg.method.fullName(".*FileOutputStream.*init.*").parameter.index(1)
```


By issuing a `reachableBy` query, various flows are found, since there are many
`FileInputStream` sinks. To reduce the number of flows, provide additional expert knowledge: that an attacker can control
the destination path to which a ZIP file is extracted by passing malicious input through the
[`getName`](https://docs.oracle.com/javase/7/docs/api/java/util/zip/ZipEntry.html#getName\(\))
method. After applying the `passes` filter on the flows, the number of flows
is reduced to `2`!

```
ocular> sink.reachableBy(source).flows.l.size
res28: Int = 1847

ocular> sink.reachableBy(source).flows.passes("getName").l.size
res29: Int = 2
```
### Resulting Flow

```
 ___________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
 | param       | type                        | method               | signature                                                                                                                                                                                             |
 |==========================================================================================================================================================================================================================================================================|
 | archiveIS(2)| java.io.InputStream         | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | archiveIS   | java.io.InputStream         | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | param1(2)   | ANY                         | <operator>.cast      | <operator>.cast                                                                                                                                                                                       |
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | zis         | java.util.zip.ZipInputStream| explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | zis         | java.util.zip.ZipInputStream| explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | this(0)     | java.util.zip.ZipInputStream| getNextEntry         | java.util.zip.ZipInputStream.getNextEntry:java.util.zip.ZipEntry()                                                                                                                                    |
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | ze          | java.util.zip.ZipEntry      | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | ze          | java.util.zip.ZipEntry      | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | ze          | java.util.zip.ZipEntry      | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | ze          | java.util.zip.ZipEntry      | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | this(0)     | java.util.zip.ZipEntry      | getName              | java.util.zip.ZipEntry.getName:java.lang.String()                                                                                                                                                     |
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | fileName2   | java.lang.String            | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | fileName2   | java.lang.String            | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | name(2)     | java.lang.String            | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | name        | java.lang.String            | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | this(0)     | java.lang.String            | substring            | java.lang.String.substring:java.lang.String(int,int)                                                                                                                                                  |
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | name        | java.lang.String            | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | name        | java.lang.String            | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | name        | java.lang.String            | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | name        | java.lang.String            | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | param1(2)   | java.lang.String            | <init>               | java.io.File.<init>:void(java.io.File,java.lang.String)                                                                                                                                               |
 | file        | java.io.File                | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | file        | java.io.File                | allocateFile         | com.google.refine.importing.ImportingUtilities.allocateFile:java.io.File(java.io.File,java.lang.String)                                                                                               |
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | file2       | java.io.File                | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | file2       | java.io.File                | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | file(2)     | java.io.File                | saveStreamToFile     | com.google.refine.importing.ImportingUtilities.saveStreamToFile:long(java.io.InputStream,java.io.File,com.google.refine.importing.ImportingUtilities$SavingUpdate)                                    |
 | file        | java.io.File                | saveStreamToFile     | com.google.refine.importing.ImportingUtilities.saveStreamToFile:long(java.io.InputStream,java.io.File,com.google.refine.importing.ImportingUtilities$SavingUpdate)                                    |
 | param0(1)   | java.io.File                | <init>               | java.io.FileOutputStream.<init>:void(java.io.File)                                                                                              
 ```

## Verifying the Vulnerability

A flow from `doPost` to `explodeArchive`
and from `explodeArchive` to a `FileOutputStream` instance is controlled. In order to verify
the existence of a vulnerability, there is only one remaining question: *is
the file actually written?* To answer this question, look at the
`write` method of the `FileOutputStream`, which is used as a sink. In addition
to the `reachableBy` query, use the `passesNot` filter, which ensures that
only flows without an `uncompressFile` method, a method that that handles `.gz` and
`.bz2`, are considered.

```
ocular> val source = cpg.method.name("explodeArchive").parameter
ocular> val sink = cpg.method.fullName(".*FileOutputStream.*write.*").parameter.index(1)
ocular> sink.reachableBy(source).passesNot("uncompressFile").p
```

### Resulting Flow

```
___________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
 | param       | type                        | method               | signature                                                                                                                                                                                             |
 |==========================================================================================================================================================================================================================================================================|
 | archiveIS(2)| java.io.InputStream         | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | archiveIS   | java.io.InputStream         | explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | param1(2)   | ANY                         | <operator>.cast      | <operator>.cast                                                                                                                                                                                       |
 | param1(2)   | ANY                         | <operator>.assignment| <operator>.assignment                                                                                                                                                                                 |
 | zis         | java.util.zip.ZipInputStream| explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | zis         | java.util.zip.ZipInputStream| explodeArchive       | com.google.refine.importing.ImportingUtilities.explodeArchive:boolean(java.io.File,java.io.InputStream,org.json.JSONObject,org.json.JSONArray,com.google.refine.importing.ImportingUtilities$Progress)|
 | stream(1)   | java.io.InputStream         | saveStreamToFile     | com.google.refine.importing.ImportingUtilities.saveStreamToFile:long(java.io.InputStream,java.io.File,com.google.refine.importing.ImportingUtilities$SavingUpdate)                                    |
 | stream      | java.io.InputStream         | saveStreamToFile     | com.google.refine.importing.ImportingUtilities.saveStreamToFile:long(java.io.InputStream,java.io.File,com.google.refine.importing.ImportingUtilities$SavingUpdate)                                    |
 | this(0)     | java.io.InputStream         | read                 | java.io.InputStream.read:int(byte[])                                                                                                                                                                  |
 | bytes       | byte[]                      | saveStreamToFile     | com.google.refine.importing.ImportingUtilities.saveStreamToFile:long(java.io.InputStream,java.io.File,com.google.refine.importing.ImportingUtilities$SavingUpdate)                                    |
 | bytes       | byte[]                      | saveStreamToFile     | com.google.refine.importing.ImportingUtilities.saveStreamToFile:long(java.io.InputStream,java.io.File,com.google.refine.importing.ImportingUtilities$SavingUpdate)                                    |
 | param0(1)   | byte[]                      | write                | java.io.FileOutputStream.write:void(byte[],int,int)                                                                                                                                                   |
```

## Conclusion

The vulnerability CVE-2018-19859 in 
OpenRefine can be detected with ShiftLeft Ocular
using `explodeArchive` and `write` (from `FileOutputStream`) as sources and
sinks, respectively. Furthermore, the existence of 
the vulnerability is validated. However, you would still need to [create an exploit to see if it is really exploitable](https://github.com/OpenRefine/OpenRefine/issues/1840). 
