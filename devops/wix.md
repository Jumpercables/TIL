# WiX Toolset (Windows Installation Packages)
## Dynamic Directory Location
In some cases, you need to install files into a `third party` software directory, and that directory location will need to be determined by the installer. This can be done by taking the following steps:
- Define a directory for the `third party` folder.

```xml
<Directory Id="TARGETDIR" Name="SourceDir">    
    <Directory Id="ThirdParty" Name="THIRDPARTYINSTALLDIR"/>
</Directory>
```

- Define a property that will contain the path to the `third party` folder that is determined by `RegistrySearch` and `DirectorySearch`.

```xml
<Property Id="THIRDPARTYINSTALLDIR">
  <!-- When installed set the [THIRDPARTYINSTALLDIR] property to the value of the InstallDir registry value. -->
  <RegistrySearch Id="ThirdPartyInstallDir" Root="HKLM" Key="SOFTWARE\Third Party" Type="raw" Name="InstallDir">   
    <!-- The DirectorySearch confirms that the directory exists, in the case that the registry is stale. -->
    <DirectorySearch Id='LocalDir' Path='[THIRDPARTYINSTALLDIR]'/>
  </RegistrySearch>
</Property>
```

- Define a custom action that will set the `third party` directory with the value of the `THIRDPARTYINSTALLDIR` property.

```xml
<!-- The custom action will set the directory named 'ThirdParty' to the [THIRDPARTYINSTALLDIR] property value. -->
<CustomAction Id="ThirdParty" Directory="ThirdParty" Value="[THIRDPARTYINSTALLDIR]"/>

<!-- Add the custom action to the install sequence, only when the directory has been located. -->
<InstallExecuteSequence>
  <Custom Action="ThirdParty" After="CostFinalize"/>
      <![CDATA[THIRDPARTYINSTALLDIR <> ""]]>
  </Custom>    
</InstallExecuteSequence>
```
