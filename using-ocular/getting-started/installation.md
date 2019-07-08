# Installing and Updating ShiftLeft Ocular

Before installing or updating ShiftLeft Ocular, make sure you have [met all requirements](../../introduction/requirements.md). In addition, for large applications, you may want to optimize ShiftLeft Ocular performance through the appropriate combination of [memory and disk sizes](ocular-memory-size.md).

**Note**: Do not make any changes to any directories, except the installation and policy directories.

Once you have installed or updated ShiftLeft Ocular, you [start ShiftLeft Ocular](starting.md).

## Installing ShiftLeft Ocular

The process of installing ShiftLeft Ocular is:

1. Unzip the Ocular distribution ZIP file provided by ShiftLeft, by issuing the following command

```
$ unzip ocular-distribution-[version].zip
```
where `[version]` is the actual version number of the distribution file.

2. Enter the password you received from ShiftLeft. The folder `ocular-distribution` is created.

3. Navigate to the folder `ocular-distribution`. From there, for Linux and MacOS X run the installer using the `./install.sh` command. For the Windows OS, run the installing using the command `.\install.ps1` and then follow the prompt.

If you don't have permission, first use the  command `chmod +x install.sh` and then run the installer again.

4. Identify where you want to install ShiftLeft Ocular (defaults to `~/bin/ocular` on Linux and MacOS X, and to `C:\Users\$USERNAME\bin\ocular` on Windows).

### Additional Configuration for Large Projects

Code analysis can require lots of memory, and unfortunately, the JVM does not pick up the available amount of memory by itself. While tuning Java memory usage is a discipline in its own right, it is usually sufficient to specify the maximum available amount of heap memory using the JVM's `-Xmx` flag. The easiest way to achieve this globally is by setting the environment variable `_JAVA_OPTS` as follows:

```
export _JAVA_OPTS="-Xmx$NG"
```
where `$N` is the amount of memory in gigabytes. You can add this line to your shell startup script, e.g., `~/.bashrc` or `~/.zshrc`.

Refer to the article [Memory Size Recommendations](ocular-memory-size.md) for more information.

## Updating ShiftLeft Ocular

You can determine your current version of ShiftLeft Ocular by using the `version` command. To update ShiftLeft Ocular:

1. Repeat steps 1 - 4 from above. 

2. Specify how you want the update to proceed. Enter either `y` to replace an existing ShiftLeft Ocular installation, `n`to not replace the existing installation, `A`to install all ShiftLeft Ocular files, `N` to install no ShiftLeft Ocular files, or `r` to rename the installation directory.
