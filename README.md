# IJava

[![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/SpencerPark/ijava-binder/master?filepath=%2Fhome%2Fjovyan%2FHelloWorld.ipynb)

A [Jupyter](http://jupyter.org/) kernel for executing Java code. The kernel executes code via the new [JShell tool](https://docs.oracle.com/javase/9/jshell/introduction-jshell.htm). Some of the additional commands should be supported in the future via a syntax similar to the ipython magics.

The kernel is currently working but there are some features that would be nice to have. There is a [TODO list](#todo) of planned features but any additional requests for new ones or prioritizing current ones are welcomed in the [issues](https://github.com/SpencerPark/IJava/issues).

If you are interested in building your own kernel that runs on the JVM check out the related project that this kernel is build on, [jupyter-jvm-basekernel](https://github.com/SpencerPark/jupyter-jvm-basekernel).

### Contents

*   [Try online](#try-online)
*   [Features](#features)
    *   [TODO](#todo)
*   [Requirements](#requirements)
*   [Installing](#installing)
*   [Configuring](#configuring)
    *   [List of options](#list-of-options)
    *   [Changing VM/compiler options](#changing-vmcompiler-options)
    *   [Configuring startup scripts](#configuring-startup-scripts)
*   [Run](#run)

### Try Online

Clicking on the [![Binder](https://mybinder.org/badge.svg)](https://mybinder.org/v2/gh/SpencerPark/ijava-binder/master?filepath=%2Fhome%2Fjovyan%2FHelloWorld.ipynb) badge at the top (or right here) will spawn a jupyter server running this kernel. The binder base is the [ijava-binder project](https://github.com/SpencerPark/ijava-binder).

### Features

Currently the kernel supports

*   Code execution
    ![output](docs/img/output.png)
*   Autocompletion (`TAB` in Jupyter notebook)
    ![autocompletion](docs/img/autocompletion.png)
*   Code inspection (`Shift-TAB` up to 4 times in Jupyter notebook)
    ![code-inspection](docs/img/code-inspection.png)
*   Colored, friendly, error message displays
    ![compilation-error](docs/img/compilation-error.png)
    ![incomplete-src-error](docs/img/incomplete-src-error.png)
    ![runtime-error](docs/img/runtime-error.png)
*   Configurable evaluation timeout
    ![timeout](docs/img/timeout.png)

#### TODO

- [ ] Display additional media types like images or audio.
- [ ] Support magics for making queries about the current environment.
- [ ] Compile javadocs when displaying introspection requests as html.
- [ ] Support loading maven dependencies at runtime.

### Requirements

1.  [Java JDK >=9](http://www.oracle.com/technetwork/java/javase/downloads/index.html). **Not the JRE**

    Ensure that the `java` command is in the PATH and is using version 9. For example:
    ```bash
    > java -version
    java version "9"
    Java(TM) SE Runtime Environment (build 9+181)
    Java HotSpot(TM) 64-Bit Server VM (build 9+181, mixed mode)
    ```

    If the kernel cannot start with an error along the lines of
    ```text
    Exception in thread "main" java.lang.NoClassDefFoundError: jdk/jshell/JShellException
            ...
    Caused by: java.lang.ClassNotFoundException: jdk.jshell.JShellException
            ...
    ```
    then double check that `java` is referring to the command for the `jdk` and not the `jre`.
    
2.  Some jupyter-like environment to use the kernel in.

    A non-exhaustive list of options:

    *   [Jupyter](http://jupyter.org/install) - main option
    *   [JupyterLab](http://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html)
    *   [nteract](https://nteract.io/desktop)
        
### Installing

After meeting the [requirements](#requirements), the kernel can be installed locally.

1.  Download the project.
    ```bash
    > git clone https://github.com/SpencerPark/IJava.git --depth 1
    > cd IJava/
    ```
2.  Build and install the kernel.
    
    On *nix `chmod u+x gradlew && ./gradlew installKernel`
        
    On windows `gradlew installKernel`

### Configuring

Configuring the kernel can be done via environment variables. These can be set on the system or inside the `kernel.json`. To find where the kernel is installed run

```bash
> jupyter kernelspec list
Available kernels:
  java           .../kernels/java
  python3        .../python35/share/jupyter/kernels/python3
```

and the `kernel.json` file will be in the given directory.

#### List of options

`IJAVA_VM_OPTS` - **default: `""`** - A space delimited list of command line options that would be passed to the `java` command if running code. For example `-Xmx128m` to set a limit on the heap size or `-ea` to enable `assert` statements. 

`IJAVA_COMPILER_OPTS` - **default: `""`** - A space delimited list of command line options that would be passed to the `javac` command when compiling a project. For example `-parameters` to enable retaining parameter names for reflection.

`IJAVA_TIMEOUT` - **default: `"-1"`** - A duration in milliseconds specifying a timeout on long running code. If less than zero the timeout is disabled.

`IJAVA_CLASSPATH` - **default: `""`** - A file path separator delimited list of classpath entries that should be available to the user code.

`IJAVA_STARTUP_SCRIPTS_PATH` - **default: `""`** - A file path seperator delimited list of `.jshell` scripts to run on startup. This includes [ijava-jshell-init.jshell](src/main/resources/ijava-jshell-init.jshell) and [ijava-magics-init.jshell](src/main/resources/ijava-magics-init.jshell).

`IJAVA_STARTUP_SCRIPT` - **default: `""`** - A block of java code to run when the kernel starts up. This may be something like `import my.utils;` to setup some default imports or even `void sleep(long time) { try {Thread.sleep(time)} catch (InterruptedException e) {}}` to declare a default utility method to use in the notebook.

#### Changing VM/compiler options

See the [List of options](#list-of-options) section for all of the configuration options.

To feed specific command line arguments to the compiler and JVM there are 2 environment variables that are checked when creating the shell.

*   `IJAVA_VM_OPTS` is the variable name for the JVM options
*   `IJAVA_COMPILER_OPTS` is the variable name for the compiler options

These variables can be assigned in the `kernel.json` by adding/editing a JSON dictionary at the `env` key.

For example to enable assertions, set a limit on the heap size, and enable parameter names in reflection:

```diff
{
  "argv": [ "java", "-jar", "{connection_file}"],
  "display_name": "Java",
- "language": "java"
+ "language": "java", 
+ "env": {
+     "IJAVA_VM_OPTS": "-ea -Xmx128m",
+     "IJAVA_COMPILER_OPTS" : "-parameter"
+ }
}
```

#### Configuring startup scripts

See the [List of options](#list-of-options) section for all of the configuration options.

To setup a startup script such as an `init.jshell` script, set the `IJAVA_STARTUP_SCRIPTS_PATH` to `init.jshell` in the `kernel.json`. This will try to execute an `init.jshell` script in the same directory as the notebook.

If desired use an absolute path to use a global init file.

```diff
{
  "argv": [ "java", "-jar", "{connection_file}"],
  "display_name": "Java",
- "language": "java"
+ "language": "java", 
+ "env": {
+     "IJAVA_STARTUP_SCRIPTS_PATH": "init.jshell"
+ }
}
```

### Run

This is where the documentation diverges, each environment has it's own way of selecting a kernel. To test from command line with Jupyter's console application run:

```bash
jupyter console --kernel=java
```

Then at the prompt try:
```java
In [1]: String helloWorld = "Hello world!"

In [2]: helloWorld
Out[2]: "Hello world!"
```
