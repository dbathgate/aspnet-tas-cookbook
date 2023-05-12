# Hosted Web Core

* [Hosted Web Core](https://learn.microsoft.com/en-us/iis/web-development-reference/native-code-development-overview/creating-hosted-web-core-applications) (HWC) is a Windows feature that allows developers to launch a standalone IIS server
* TAS uses an [HWC wrapper written in GoLang](https://github.com/cloudfoundry/hwc) to run IIS applications inside a container
* It is possible to use this same HWC wrapper locally to do compatibility testing for TAS without having to deploy to TAS

## Enable Windows Features

* Open powershell as **Administrator** and run the following:
```powershell
Enable-WindowsOptionalFeature -Online -All -FeatureName IIS-WebServer
Enable-WindowsOptionalFeature -Online -All -FeatureName IIS-WebSockets
Enable-WindowsOptionalFeature -Online -All -FeatureName IIS-HostableWebCore
Enable-WindowsOptionalFeature -Online -All -FeatureName IIS-ASPNET45
```

## Download the HWC wrapper from github

* [Download from here](https://github.com/cloudfoundry/hwc/releases)
* **NOTE**: `hwc.exe` is the standard version, but `hwc_x86.exe` may be required if app has 32-bit dependencies

## Install HWC to your Environment Variable PATH
* [Example here](https://gist.github.com/ScribbleGhost/752ec213b57eef5f232053e04f9d0d54)

## Running HWC

* Open a PowerShell terminal as **Administrator**
* Find the directory where the `web.config` exists
* Pick a port that is not currently in use, try `8080`
* Run the following command with the port and path to the directory
```powershell
&{ $env:PORT=8082; hwc.exe -appRootPath "C:\workspace\OnboardingApp\OnboardingApp\"}
```
* Alternatively
```powershell
 cd C:\workspace\OnboardingApp\OnboardingApp\
 &{ $env:PORT=8082; hwc.exe}
 ```
 * Navigate with a browser to [http://localhost:8080](http://localhost:8080)

 ## Stopping HWC
 * In the terminal running HWC, press Control+C twice

## Troubleshooting

### Incorrect format error

* If you see the following error when attempting to view the app in a browser
```
Could not load file or assembly 'OnboardingApp' or one of its dependencies. An attempt was made to load a program with an incorrect format.
```
* Then your app likely has 32-bit dependencies
* Switch to using `hwc_x86.exe`
* **NOTE**: This will also have to be changed in `manifest.yml`
```yaml
applications:
- name: OnboardingApp
  stack: windows
  buildpack:
  - hwc_buildpack
  command: .\.cloudfoundry\hwc_x86.exe
```

### Error Code: `0x800700b7`

* If HWC returns the following error in the terminal
```
HWC Failed to start: return code: 0x800700b7
```
* Then that port is already in use, switch to a different port and try again

### Error Code: `0x80070005`
* If HWC returns the following error in the terminal
```
HWC Failed to start: return code: 0x80070005
```
* Then you are not running HWC as Administrator, open an Administrator powershell and try again