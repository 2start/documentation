# Using ShiftLeft in Interactive and Non-Interactive Modes

ShiftLeft Ocular lets you perform code analysis on CPGs and Security Profiles, both interactively using a REPL or with non-interactive scripts. 

## Using a REPL

A REPL is an interactive shell with support for the Ocular Query Language (OQL). Among other features, a REPL offers utilities for exporting security analysis results as plain text or in JSON format, readline support and tab-completion (included in the ShiftLeft Ocular trial version).

The ShiftLeft Ocular underlying shell is an interactive Scala shell which includes useful things like:

* \\&lt;TAB&gt; for autocomplete.
* \\&lt;UP&gt; and <DOWN> for moving through the command history.
* \\&lt;CTRL-r&gt; to search the command history.
* `helpMsg` and `status`.
     
## Using Non-Interactive Scripts

ShiftLeft Ocular can be used in non-interactive mode, to execute commands and operations without typing them after the prompt. The commands are stored in a file which can be specified as an argument. ShiftLeft Ocular runs those commands and then exits. For example, include in `test.sc`

```
@main def exec(cpgFile: String, outFile: String) = {
   loadCpg(cpgFile)
   cpg.namespace.name.l |> outFile
}

```
You can include arbitrary Scala code in `test.sc` and use the `|>`
operator to pipe output into files. The script is run as 

```
./ocular.sh --script test.sc --params cpgFile=/fullpath/to/cpg.bin.zip,outFile=out.log
```

### Writing JSON and Pretty-Printed JSON

To write JSON

```
cpg.method.toJson |> "/tmp/foo" 
```

To write Pretty-Printed JSON

```
cpg.method.toPrettyJson |> "/tmp/foo"
```

### Appending to Files

To append to a JSON file

```
cpg.method.toJson ||> "/tmp/foo" 
```

To append to a Pretty-Printed JSON file

```
cpg.method.toPrettyJson ||> "/tmp/foo"
```