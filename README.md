# Installing wixl
```bash
sudo apt install wixl
```
For Conditional setup using Feature, use distro with wixl v0.106. Ubuntu 25.10 can install wixl v0.106 by default. Or build your own from https://download.gnome.org/sources/msitools/0.106/
- `Condition` tag was added in msitools v0.106 https://download.gnome.org/sources/msitools/0.106/
  ```
  v0.106
  ======
  - !75 Support <Condition> inside <Feature> and <Component>
  ```

# Building MSI
```bash
wixl -D VERSION=18.5.0 -o gitlab-runner.msi wixl-gitlab-runner.wxs
```

# Silent setup MSI
Launch elevated CMD and paste modified command
```powershell
msiexec /qn /i "gitlab-runner.msi" GITTOKEN=Token123 GITURL=http://github.com GITNAME=server1
```

# TODO
- `CustomActions` not working properly
  - replaced with modification of `config.toml` using `IniFile`

# Passing arguments to .msi package
Create `Property` with `Id`="GITTOKEN", set default `Value` in case you don't provide launch argument when starting .msi
```xml
<Wix xmlns="http://wixtoolset.org/schemas/v4/wxs">
  <Package Id="GitLab-Runner.Msi" Name="GitLab Runner" Manufacturer="GitLab" Version="18.14.0">
    <Property Id="GITTOKEN"  Value="someDefaultToken" />
```
Pass your data to `GITTOKEN` property as launch argument:
- PowerShell token with spaces
  ```powershell
  msiexec /i "gitlab-runner.msi" GITTOKEN=`"Token with spaces`" GITURL=http://github.com
  ```
- CMD token with spaces
  ```cmd
  msiexec /i "gitlab-runner.msi" GITTOKEN="Token with spaces" GITURL=http://github.com
  ```

# Creating/changing config.toml
- Do not use a closed bracket as part of the section name

  | TOML section | wixl section name |
  | - | - |
  | [[runners]] |	[runners |