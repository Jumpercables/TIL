# PowerShell
## Double-click to Run.
You can't double-click to run `.PS1` files, but you can execute a `.BAT` file that way. You can accomplish the same behavior by having a batch file call the PowerShell script from the command line.

```
@ECHO OFF
PowerShell.exe -Command "& '%~dpn0.ps1'"
PAUSE
```

> The batch file will need to be placed in the same folder as your PowerShell script and have the same file name. So if your PowerShell script is called Test.ps1", you'll want to name your batch file Test.bat" and make sure it's in the same folder.

## Without Administrator Access
The -ExecutionPolicy parameter can be used to modify the ExecutionPolicy that is used when you spawn a new PowerShell session.

```
@ECHO OFF
PowerShell.exe -NoProfile -ExecutionPolicy Bypass -Command "& '%~dpn0.ps1'"
PAUSE
```

## With Administrator Access
Use the `-Verb RunAs` in its arguments for the `Start-Process` to launch an application with Administrator permissions.

```
@ECHO OFF
PowerShell.exe -NoProfile -Command "& {Start-Process PowerShell.exe -ArgumentList '-NoProfile -ExecutionPolicy Bypass -File ""%~dpn0.ps1""' -Verb RunAs}"
PAUSE
```

## Store Credentials in an XML file
Using the `Export-Clixml` and `Import-Clixml` you can store away the entire `System.Management.Automation.PSCredential` Object as an XML file.   

### Export
The credentials can be piped to the `Export-Clixml` method.

```
param (        
    [string]$Path = (Resolve-Path .\).Path    
 )

$Credentials = Get-Credential –Credential ""
$Credentials | Export-Clixml $Path\Credential.xml
```

### Import-Clixml
The XML of the credentials can be loaded using the `Import-Clixml` method.

```
param (        
    [string]$Path = (Resolve-Path .\).Path        
 )

$Credentials = Import-Clixml $Path\Credential.xml
```
