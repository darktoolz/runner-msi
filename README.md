# Building
```bash
dotnet build --configuration Release
```

# Silent setup
Launch elevated PowerShell and paste modified command
```powershell
msiexec /qn /i "bin\Release\runner.msi" GITTOKEN=`"Token123 with spaces`" GITURL=http://github.com
```
Launch elevated CMD and paste modified command
```cmd
msiexec /qn /i "bin\Release\runner.msi" GITTOKEN="Token with spaces" GITURL=http://github.com
```

# Passing arguments to .msi package
Create `Property` with `Id`="GITTOKEN", set default `Value` in case you don't provide launch argument when starting .msi
```xml
<Wix xmlns="http://wixtoolset.org/schemas/v4/wxs">
  <Package Id="Darktooz.Runner.Msi" Name="Darktooz GitLab Runner" Manufacturer="Darktooz" Version="0.0.2">
    <Property Id="GITTOKEN"  Value="someDefaultValue" />
```
Pass your data to `GITTOKEN` property as launch argument
```powershell
msiexec /i "bin\Release\runner.msi" GITTOKEN=`"Token with spaces`" GITURL=http://github.com
```
```cmd
msiexec /i "bin\Release\runner.msi" GITTOKEN="Token with spaces" GITURL=http://github.com
```
