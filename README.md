# Azure DevOps Agents for BC Examples

Some pipelines need Variables. Here are scripts to create Variable Groups:

1) [Setup Pipeline Variable Groups](./setup-pipeline-variables.md)

Examples for installation of Azure DevOps Agents / Deployment Agents for BC with PowerShell:

1) [Setup Build Agents](./setup-build-agents.md)
1) [Setup Deployment Agents](./setup-deployment-agents.md)

Examples to simulate Staging Environments for D365BC with Docker Containers:

1) [Create BC-DockerContainer for Staging Simulation](./create-bc-containers-for-staging.md)

## Related Background Information

Here are some articles related to these PS-Scripts:

1) [Artifacts & Deployment Stages (Part I)](https://www.linkedin.com/pulse/ci-cd-dynamics-365-business-central-part-i-michael-megel/)
1) [Deployment Flow & Triggers (Part II)](https://www.linkedin.com/pulse/ci-cd-dynamics-365-business-central-part-ii-michael-megel/)
1) [Build & Deployment Agents (Part III)](https://www.linkedin.com/pulse/ci-cd-dynamics-365-business-central-part-iii-michael-megel/)

## Development Environment

The best development environment is nothing without tools! Here are some PowerShell Scripts to install my preferred selection. *Admin Rights may be needed to execute these Scripts.*

### Chocolatey & Tools

Install from **PowerShell**:

* Chocolatey [https://chocolatey.org](https://chocolatey.org)
* [Git](https://chocolatey.org/packages/git)
* [UPack (Universal Package)](https://chocolatey.org/packages/upack)
* [Azure-CLI](https://chocolatey.org/packages/azure-cli) + [Azure-CLI Extension for Azure DevOps](https://marketplace.visualstudio.com/items?itemName=ms-vsts.cli)
* [Google Chrome](https://chocolatey.org/packages/GoogleChrome)

```powershell
# Install Choco
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
# Install Git
choco install -y git
choco upgrade -y git
# Install UPack
choco install -y upack
choco upgrade -y upack
# Install Azure CLI
choco install -y azure-cli 
choco upgrade -y azure-cli
# Install Google Chrome
choco install -y googlechrome
choco upgrade -y googlechrome
# Reload the PATH variable
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
# Install Azure CLI Extension for Azure-DevOps
az extension add --name azure-devops
```

### Chocolatey & VSCode & VSCode Extensions for AL

```powershell
# Install Choco
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
# Install VSCode
choco install -y vscode
choco upgrade -y vscode
# Reload the PATH variable
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")
# Install VS-Code Extensions for AL
try { code --install-extension file-icons.file-icons | Out-Null } catch {}
try { code --install-extension andrzejzwierzchowski.al-code-outline | Out-Null } catch {}
try { code --install-extension rasmus.al-formatter | Out-Null } catch {}
try { code --install-extension martonsagi.al-object-designer | Out-Null } catch {}
try { code --install-extension davidanson.vscode-markdownlint | Out-Null } catch {}
try { code --install-extension hnw.vscode-auto-open-markdown-preview | Out-Null } catch {}
try { code --install-extension eriklynd.json-tools | Out-Null } catch {}
try { code --install-extension waldo.crs-al-language-extension | Out-Null } catch {}
try { code --install-extension donjayamanne.githistory | Out-Null } catch {}
try { code --install-extension eamodio.gitlens | Out-Null } catch {}
try { code --install-extension heaths.vscode-guid | Out-Null } catch {}
try { code --install-extension streetsidesoftware.code-spell-checker | Out-Null } catch {}  
```
