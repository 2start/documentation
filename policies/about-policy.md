# About ShiftLeft Policies

Policies are used by ShiftLeft Inspect and ShiftLeft Ocular to identify how your application communicates with the outside world, which transformations exist on data, and which information flows should be considered security violations. Security Profiles, which automate code analysis by summarizing the vulnerabilities and data leaks present in code, are derived from Policies.

ShiftLeft includes default Policies that specify the most common types of vulnerabilities. In addition, ShiftLeft Ocular users can [create custom Policies](custom-policy.md) to introduce additional knowledge about, and to exclude parts of a default Policy that does not apply to, your application.

Policies are written in the ShiftLeft [Policy Language](policy-language.md) and are identified by the filename extension `.Policy`. Policies are stored under organization-bound domains.

## Policies and ShiftLeft Ocular

When you [create and work with a CPG using ShiftLeft Ocular](../using-ocular/getting-started/create-cpg.md), policies are automatically loaded and assigned.

ShiftLeft Ocular Policies are located in the directory

```
~/bin/ocular/policy
```

## Policies and ShiftLeft Inspect

The default or assigned Policy is automatically loaded and applied when ShiftLeft Inspect uses your application's [Code Property Graph (CPG)](../introduction/understanding-cpg.md). ShiftLeft infers the correct Policy to use, based on the source language. By default, Policies use a generic set of sensitive variable names, which directly affect the security results you see. 

ShiftLeft Inspect Policies are located in the directory

```
~/.shiftleft/policy/
```

### The ShiftLeft Inspect Default Policy

The default Policy for ShiftLeft Inspect contains directives that define and identify:

* Exposed methods, interface interactions and transformations.

* Information flows, particularly data leaks.

* Taint semantics. 

To illustrate the default Policy, take a look at how  deserialization vulnerabilities are identified. The following directive

```
// [~/.shiftleft/policy/dynamic/java/io/ObjectInputStream.policy:]

IO deserializer = METHOD -f "java.io.ObjectInputStream.readObject:java.lang.Object()" { INST "SINK" }
```

specifies that the instance parameter of the method `readObject` should be considered as a data sink of a deserializer, that is, the instance parameter is deserialized.

This directive determines that all parameters tagged with the Spring annotations `CookieValue`, `PathVariable`, and a few others are to be tagged as "attacker-controlled". 

```
// ~/.shiftleft/policy/dynamic/org/springframework/exposed.policy:

TAG "DATA_TYPE" -v "attacker-controlled" METHOD -a r"org\.springframework\.web\.bind\.annotation\.(Request|Get|Post|Put|Delete|Patch)Mapping" PAR -a r"org\.springframework\.web\.bind\.annotation\.(CookieValue|PathVariable|RequestParam|RequestBody)"
```

And this directive specifies that flows of attacker-controlled data into deserializers are worth reporting

```
// ~/.shiftleft/policy/static/execute.policy:

CONCLUSION attacker-to-deserializer = FLOW DATA (attacker-controlled) -> IO (deserializer)
WHEN CONCLUSION attacker-to-deserializer => EMIT {
    title: "Attacker controlled data to deserialization",
    description: "Attacker controlled data is deserialized in this flow. ...",
    category: "a1-injection",
    score: "8.0"
}
```

In this directive, the Policy also allows data transformations and checks to be identified in order to report flows of data, for data that does not undergo validation. 

Refer to the ShiftLeft [Policy Language](policy-language.md) for additional information and examples.

### Working with Policies with ShiftLeft Inspect

Using the ShiftLeft Command Line Interface (CLI), you can

* View a Policy.
* List the default Policy assignment.
* Assign another Policy as the default.
* Delete the default Policy assignment.

Refer to the [CLI Reference](../using-inspect-protect/using-cli/cli-reference.md) for information.
