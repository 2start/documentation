# `CpgLoader` Class

Existing Code Property Graphs (CPGs) and their associated default layers (overlays) and indexes are loaded into your workspace using the `CpgLoader` class. This class is defined in the package `io.shiftleft.codepropertygraph.cpgloading`. `CpgLoader` provides the following methods:

* `load`. Loads CPGs.
* `addOverlays`. Loads layers and adds them to the active CPG.
* `createIndex`. Create indexes to increase the speed of subsequent queries

## `CpgLoader.load`

The method 'CpgLoader.load' loads the CPG of the target application(s).

For example, the following Scala command loads a CPG using `CpgLoader`, makes it available to the variable `cpg`, and queries the CPG to list all methods in your application code:

```scala
ocular> val cpg = CpgLoader.load(<filename>)
cpg.method.fullName.l
```

where `filename` is the absolute location of the target application, for example "cpg.bin.zip".
  
`CpgLoader.load` can be configured using the method [`CpgLoaderConfig`](cpgloaderconfig.md).

## `CpgLoader.addOverlays`

The method `CpgLoader.addOverlays` loads a layer and adds it to the active CPG. 

For example, use the following Scala code to load both the CPG and its Security Profile layer 

```scala
ocular> val cpg = CpgLoader.load(filename)
CpgLoader.addOverlays(Seq(filename), cpg)
```
where `filename` is the absolute location of the layer, for example "sp.bin.zip".
  
## `CpgLoader.createIndexes`

If you make large changes to a CPG after loading, but before querying it, then you may want to delay the creation of indexes. You can achieve this with the following example Scala code

```scala
ocular> val cpg = CpgLoader.load(filename, CpgLoaderConfig.default.doNotCreateIndexesOnLoad)
// ... modify `cpg`
CpgLoader.createIndexes()
```