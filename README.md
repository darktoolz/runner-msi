# Installing wixl
```bash
sudo apt install wixl
```

# Building MSI
```bash
wixl -o gitlab-runner.msi wixl-gitlab-runner.wxs
```

# Silent setup MSI
Launch elevated CMD and paste modified command
```cmd
msiexec /qn /i "bin\Release\runner.msi" GITTOKEN=Token123 GITURL=http://github.com
```

# TODO
- `CustomActions` not working properly
- `Condition` tag is absent in `wixl` - 32 and 64 bit combined package is impossible

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