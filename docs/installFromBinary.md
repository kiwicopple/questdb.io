---
id: installFromBinary
title: Install from Binary
---

This section describes how to [setup](#setup) and [use](#using-questdb) QuestDB using the binaries.

## Requirements

- 64-bit MacOS, Windows or Linux.
- Oracle Java JRE or JDK 8 and above, or OpenJDK equivalent.
> OpenJDK may incur a performance penalty of up to 20% as it contains fewer intrinsics than the Oracle counterpart.

## Setup

#### Step 1 - Download & install JAVA
If you already have a suitable version JAVA installed, you can skip this step.
You can find the package corresponding to your architecture on the <a href="https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html" target="_blank">Oracle Download page</a>.


#### Step 2 - Download QuestDB
You can download QuestDB from our <a href="/getstarted" target="_blank">download page</a>.
To install, simply extract the files into the directory of your choice. 

> Throughout this documentation, we refer to this directory as the `installation_directory`. When starting, QuestDB requires 
>another directory where the configuration, logs, and data files are saved. We call this directory the [root_directory](rootDirectoryStructure.md).


## Using QuestDB
QuestDB comes with an executable `questdb.exe` for Windows, and script `questdb.sh` for MacOS and Linux which can 
be used to control QuestDB as a service. On Windows, QuestDB can also be [started interactively](#use-interactively-windows).

Navigate to the `installation_directory`.
```sql
cd installation_directory
```
You can interact with the service using the following syntax and commands.

<!--DOCUSAURUS_CODE_TABS-->
<!--Linux & MacOS syntax-->
```sql
./questdb.sh [start|stop|status] [-d dir] [-f] [-t tag] 
```
<!--Windows syntax-->
```sql
questdb.exe [start|stop|status|install|remove] [-d dir] [-f] [-j JAVA_HOME] [-t tag] 
```
<!--END_DOCUSAURUS_CODE_TABS-->

|Command | Description |
|-----|------|
|[start](#start)| Starts Windows service. Default service name is `QuestDB`  |
|[stop](#stop) | Stops Windows service |
| [status](#status) | Shows service status. This command is useful for troubleshooting problems with the service. It prints `RUNNING` or `INACTIVE` if the service is start or stopped respectively |
| [install](#install) | Install the Windows service |
| [remove](#remove) | Remove the Windows service |


### Start 
`start` - starts the questdb service.

<!--DOCUSAURUS_CODE_TABS-->
<!--Linux & MacOS-->
```sql
./questdb.sh start
```
<!--Windows-->
```sql
questdb.exe start
```
<!--END_DOCUSAURUS_CODE_TABS-->

#### Behaviour
QuestDB will start and run in the background and continue running even if you close the session. You will need to actively [stop it](#stop).


#### Default directories
By default, QuestDB `root_directory` will be the following
<!--DOCUSAURUS_CODE_TABS-->
<!--Linux -->
```shell script
$HOME/.questdb
```
<!--MacOS -->
```shell script
/usr/local/var/questdb/
```
<!--Windows-->
```shell script
C:\Windows\System32\questdb
```
<!--END_DOCUSAURUS_CODE_TABS-->


#### Options
- `-d` - specify QuestDB's `root_directory`. 
- `-f` - force reload the web console. The web console is cached otherwise and the HTML page will not be reloaded automatically in case it has been changed.
- `-j (Windows only)` - path to JAVA_HOME
- `-t` - specify a service tag. You can use this option to run several services and administer them separately.

##### Examples
<!--DOCUSAURUS_CODE_TABS-->
<!-- -d Linux-->
```sql
./questdb.sh start -d '/home/user/my_new_root_directory'
```
<!-- -d Windows-->
```sql
questdb.exe start -d 'C:\Users\user\my_new_root_directory'
```
<!-- -j -f Windows -->
```sql
questdb.exe start -j 'C:\Program Files\Java\jdk1.8.0_141'
```
<!-- -d -j -t Windows -->
```sql
questdb.exe start -d 'C:\Users\user\my_new_root_directory' -j 'C:\Program Files\Java\jdk1.8.0_141' -t 'mytag' 
```
<!--END_DOCUSAURUS_CODE_TABS-->

### Stop
`stop` - stops the default `questdb` service, or the service specified with the `-t` option.

#### Examples
<!--DOCUSAURUS_CODE_TABS-->
<!--Linux & MacOS (default)-->
```sql
./questdb.sh stop
```
<!--Windows (default)-->
```sql
questdb.exe stop
```
<!--Linux & MacOS (specific tag)-->
```sql
./questdb.sh stop -t 'my-questdb-service'
```
<!--Windows (specific tag)-->
```sql
questdb.exe stop -t 'my-questdb-service'
```
<!--END_DOCUSAURUS_CODE_TABS-->


### Status
`status` shows service status. This command is useful for troubleshooting problems with the service. It prints `Running` or `Not running` if the service is start or stopped respectively. On Unix systems, it also prints the `PID`

#### Examples
<!--DOCUSAURUS_CODE_TABS-->
<!--Linux & MacOS (default)-->
```sql
./questdb.sh status
```
<!--Windows (default)-->
```sql
questdb.exe status
```
<!--Linux & MacOS (specific tag)-->
```sql
./questdb.sh status -t 'my-questdb-service'
```
<!--Windows (specific tag)-->
```sql
questdb.exe status -t 'my-questdb-service'
```
<!--END_DOCUSAURUS_CODE_TABS-->

### Install
`install` - installs the Windows questdb service. It will start automatically at startup.

> `install` is only available on Windows.

#### Examples
<!--DOCUSAURUS_CODE_TABS-->
<!--Install (default)-->
```sql
questdb.exe install
```
<!--Install (specific tag)-->
```sql
questdb.exe install -t 'my-questdb-service'
```
<!--END_DOCUSAURUS_CODE_TABS-->

### Remove
`remove` - removes the Windows questdb service. It will no longer start at startup.


> `remove` is only available on Windows.

#### Examples
<!--DOCUSAURUS_CODE_TABS-->
<!--Remove (default)-->
```sql
questdb.exe remove
```
<!--Remove (specific tag)-->
```sql
questdb.exe remove -t 'my-questdb-service'
```
<!--END_DOCUSAURUS_CODE_TABS-->

## Use interactively (Windows)
You can start QuestDB interactively by running `questdb.exe`.

#### Behaviour 
This will launch QuestDB interactively in the active `Shell` window. QuestDB will be stopped when the Shell is closed.

#### Default directory
When started interactively, QuestDB's root directory defaults to the `current` directory.

To start, run the following.
```sql
questdb.exe
```

To stop, simply press <kbd>Ctrl</kbd>+<kbd>C</kbd> in the `Shell` window or close it.

