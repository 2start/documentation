# Querying the CPG

A custom query is a query that the user writes, as opposed to the default queries in a Policy.

You can run a query on either the active CPG or on all CPGs in your workspace.

combine queries from multiple CPGs by using cpgs.flatMap{cpg => cpg.method.l }


You use the Ocular Query Language (OQL) to [formulate queries to scan for and identify data flow vulnerabilities](../using-ocular/common-queries/data-flows.md). These queries can be automatically executed using a [non-interactive script](modes.md). However, a better approach is to create a custom Policy. Policies define methods that introduce data into the application, sensitive operations, and data flow.


## Querying the Active CPG

The active CPG is the CPG most recently loaded into your workspace. You query the active CPG using the command 
`cpg`, for example 

`ocular> cpg.method.fullName.l`

## Querying all CPGs in Your Workspace

To run a query on all CPGs in your workspace and join results, use the command `cpgs`, for example

`ocular> cpgs.flatMap(_.method.fullName.l)`

## Browsing Query Results

If your query results are extensive, you can use the following command to open a less-like application (pager) and scroll through the results

```
browse(<query>)
```

where <query> is the actual query whose results you want to scroll.

## Writing Query Results to a File

ShiftLeft Ocular uses `.l` (**not** `.p`) to list query results.

To write query results to a file, use either of the following commands, where <query> is the actual query whose results you want to output, for example `cpg.namespace.name`.
  
To write results and create a new file at the same time, use

```
<query>.l |> "out.txt"
```


To append results to an already existing file, use

```
<query>.l |>> "out.txt"
```
