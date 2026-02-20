# taskkill
## Syntax

Copy

```
taskkill [/s <computer> [/u [<domain>\]<username> [/p [<password>]]]] {[/fi <filter>] [...] [/pid <processID> | /im <imagename>]} [/f] [/t]
```

[](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/taskkill#parameters)

### Parameters

Expand table

|Parameter|Description|
|---|---|
|/s `<computer>`|Specifies the name or IP address of a remote computer (do not use backslashes). The default is the local computer.|
|/u `<domain>\<username>`|Runs the command with the account permissions of the user who is specified by `<username>` or by `<domain>\<username>`. The **/u** parameter can be specified only if **/s** is also specified. The default is the permissions of the user who is currently logged on to the computer that is issuing the command.|
|/p `<password>`|Specifies the password of the user account that is specified in the **/u** parameter.|
|/fi `<filter>`|Applies a filter to select a set of tasks. You can use more than one filter or use the wildcard character (`*`) to specify all tasks or image names. The valid filters are listed in the **Filter names, operators, and values** section of this article.|
|/pid `<processID>`|Specifies the process ID of the process to be terminated.|
|/im `<imagename>`|Specifies the image name of the process to be terminated. Use the wildcard character (`*`) to specify all image names.|
|/f|Specifies that processes be forcefully ended. This parameter is ignored for remote processes; all remote processes are forcefully ended.|
|/t|Ends the specified process and any child processes started by it.|

[](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/taskkill#filter-names-operators-and-values)

#### Filter names, operators, and values

Expand table

|Filter Name|Valid Operators|Valid Value(s)|
|---|---|---|
|STATUS|eq, ne|`RUNNING \| NOT RESPONDING \| UNKNOWN`|
|IMAGENAME|eq, ne|Image name|
|PID|eq, ne, gt, lt, ge, le|PID value|
|SESSION|eq, ne, gt, lt, ge, le|Session number|
|CPUtime|eq, ne, gt, lt, ge, le|CPU time in the format _HH:MM:SS_, where _MM_ and _SS_ are between 0 and 59 and _HH_ is any unsigned number|
|MEMUSAGE|eq, ne, gt, lt, ge, le|Memory usage in KB|
|USERNAME|eq, ne|Any valid user name (`<user>` or `<domain\user>`)|
|SERVICES|eq, ne|Service name|
|WINDOWTITLE|eq, ne|Window title|
|MODULES|eq, ne|DLL name|

[](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/taskkill#remarks)

## Remarks

- The **WINDOWTITLE** and **STATUS** filters aren't supported when a remote system is specified.
    
- The wildcard character (`*`) is accepted for the `*/im` option, only when a filter is applied.
    
- Ending a remote process is always carried out forcefully, regardless whether the **/f** option is specified.
    
- Providing a computer name to the hostname filter causes a shutdown, stopping all processes.