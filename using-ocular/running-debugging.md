# Running and Debugging ShiftLeft Ocular

Use the following command to run ShiftLeft Ocular: 

```bash
./ocular.sh
```

Once started, ShiftLeft Ocular provides the following commands.

* ***loadCpg(filename).*** Loads the CPG stored
     at `filename`. The format is inferred from the file
     extension. Valid extensions are ".xml" for GraphML and ".bin.zip"
     for the default binary format (recommended). The CPG is made
     available via the object `cpg`.
     
* ***loadSp(filename, isJson=true).*** Loads the Security Profile 
     stored at `filename`. The Security Profile is expected to be in the default
     binary format, or in its JSON version if `isJson` is explicitly
     set to true. The Security Profile is made available via the object `sp`.

## Unix Powertools

Currently only work with `List`. If you need these commands extended or changed, [contact us](https://www.shiftleft.io/contact/).

* pipe output to outfile: `cpg.namespace.name.l |> "out.txt"`
* append to outfile: `cpg.namespace.name.l |>> "out.txt"`

## Ammonite Tricks

* To print the data structure, use `.l` (don't use `.p`)
* To open a less-like application, use `browse(yourquery)` 


## Debugging ShiftLeft Ocular with jdb

```bash
export JAVA_OPTS='-Xdebug -Xrunjdwp:transport=dt_socket,address=8002,server=y,suspend=n'
./ocular.sh
```
The first line of output is `Listening for transport dt_socket at address: 8002`. You can now connect to remote debug with your favorite debugger, e.g. Intellij or jdb command line.