# Building
```bash
cd v1
dotnet build --configuration Release
```

# Silent setup
Launch elevated PowerShell and paste modified command
```powershell
msiexec /qn /i "bin\Release\runner.msi" GITTOKEN=Token123 GITURL=http://github.com
```
Launch elevated CMD and paste modified command
```cmd
msiexec /qn /i "bin\Release\runner.msi" GITTOKEN=Token123 GITURL=http://github.com
```

# Passing arguments to .msi package
Create `Property` with `Id`="GITTOKEN", set default `Value` in case you don't provide launch argument when starting .msi
```xml
<Wix xmlns="http://wixtoolset.org/schemas/v4/wxs">
  <Package Id="Darktoolz.Runner.Msi" Name="Darktoolz GitLab Runner" Manufacturer="Darktoolz" Version="0.0.2">
    <Property Id="GITTOKEN"  Value="someDefaultValue" />
```
Pass your data to `GITTOKEN` property as launch argument:
- PowerShell token with spaces
```powershell
msiexec /i "bin\Release\runner.msi" GITTOKEN=`"Token with spaces`" GITURL=http://github.com
```
- CMD token with spaces
```cmd
msiexec /i "bin\Release\runner.msi" GITTOKEN="Token with spaces" GITURL=http://github.com
```

# 2 versions of Service setup

## V1 - WiX Service manipulations
You still need CustomAction to register gitlab-runner
```xml
<ServiceInstall Name="[SERVICENAME]" DisplayName="$(ProductName)" Description="Darktoolz GitLab Runner" Type="ownProcess" Start="auto" ErrorControl="normal" Vital="no" Interactive="no"
    Arguments="run --config &quot;[INSTALLFOLDER]bin\config.toml&quot; --working-directory &quot;[INSTALLFOLDER]bin&quot; --service [SERVICENAME] --syslog" />
<ServiceControl Name="[SERVICENAME]" Remove="uninstall" Start="install" Stop="both" Wait="yes" />
<ServiceConfig ServiceName="[SERVICENAME]" OnInstall="yes" DelayedAutoStart="yes" />
<CustomAction Id="Register_Runner_x64" Directory="AppBinDirectory" ExeCommand="[AppBinDirectory]gitlab-runner-windows-amd64.exe register --non-interactive --url &quot;[GITURL]&quot; --token &quot;[GITTOKEN]&quot; --executor &quot;docker-windows&quot; --docker-image mcr.microsoft.com/windows/servercore:1809_amd64" Execute="deferred" Return="ignore" Impersonate="no" />
<InstallExecuteSequence>
    <Custom Action="Register_Runner_x64" Before="InstallFinalize" Condition="VersionNT64 AND (Not Installed)" />
</InstallExecuteSequence>
```

## V2 - Custom actions
https://gitlab-docs-d6a9bb.gitlab.io/runner/install/windows.html
- Register gitlab-runner on .MSI install
```powershell
.\gitlab-runner.exe register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --token "$RUNNER_TOKEN" \
  --executor "docker-windows" \
  --docker-image mcr.microsoft.com/windows/servercore:1809_amd64 \
  --description "docker-runner"
```
- Run gitlab-runner service using Built-in System Account on .MSI install
```powershell
.\gitlab-runner.exe install
.\gitlab-runner.exe start
```
- Uninstall gitlab-runner service on .MSI uninstall
```powershell
.\gitlab-runner.exe stop
.\gitlab-runner.exe uninstall
```

```xml
<CustomAction Id="Register_Runner_x64" Directory="AppBinDirectory" ExeCommand="[AppBinDirectory]gitlab-runner-windows-amd64.exe register --non-interactive --url &quot;[GITURL]&quot; --token &quot;[GITTOKEN]&quot; --executor &quot;docker-windows&quot; --docker-image mcr.microsoft.com/windows/servercore:1809_amd64" Execute="deferred" Return="ignore" Impersonate="no" />
<CustomAction Id="Install_Runner_x64" Directory="AppBinDirectory" ExeCommand="[AppBinDirectory]gitlab-runner-windows-amd64.exe install" Execute="deferred" Return="ignore" Impersonate="no" />
<CustomAction Id="Start_Runner_x64" Directory="AppBinDirectory" ExeCommand="[AppBinDirectory]gitlab-runner-windows-amd64.exe start" Execute="deferred" Return="ignore" Impersonate="no" />
<CustomAction Id="Stop_Runner_x64" Directory="AppBinDirectory" ExeCommand="[AppBinDirectory]gitlab-runner-windows-amd64.exe stop" Execute="deferred" Return="ignore" Impersonate="no" />
<CustomAction Id="Uninstall_Runner_x64" Directory="AppBinDirectory" ExeCommand="[AppBinDirectory]gitlab-runner-windows-amd64.exe uninstall" Execute="deferred" Return="ignore" Impersonate="no" />
<InstallExecuteSequence>
    <Custom Action="Stop_Runner_x64" Before="RemoveFiles" Condition="Installed AND (REMOVE = &quot;ALL&quot;)" />
    <Custom Action="Uninstall_Runner_x64" Before="RemoveFiles" Condition="Installed AND (REMOVE = &quot;ALL&quot;)" />
    <Custom Action="Register_Runner_x64" Before="InstallFinalize" Condition="VersionNT64 AND (Not Installed)" />
    <Custom Action="Install_Runner_x64" Before="InstallFinalize" Condition="VersionNT64 AND (Not Installed)" />
    <Custom Action="Start_Runner_x64" Before="InstallFinalize" Condition="VersionNT64 AND (Not Installed)" />
</InstallExecuteSequence>
```