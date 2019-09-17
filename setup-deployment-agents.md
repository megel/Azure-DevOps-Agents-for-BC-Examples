# Setup Deployment-Agents

## Prerequisites

1) Get your [VSTS-Token](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops)
1) Install [AzuredevOpsAPIUtils](https://www.powershellgallery.com/packages/AzureDevOpsAPIUtils) with PowerShell:

```PowerShell
Install-Module AzuredevOpsAPIUtils -Force
Update-Module  AzuredevOpsAPIUtils
Import-Module  AzuredevOpsAPIUtils -Force
```

## Create Azure DevOps Deployment Pool

Create an Azure DevOps [Deployment Pool](https://docs.microsoft.com/en-us/azure/devops/pipelines/release/deployment-groups/?view=azure-devops) for your organization with PowerShell:

```PowerShell
$poolName       = "<POOL-NAME>"
$devOpsURL      = "https://dev.azure.com/<YOUR-ORGANIZATION>"
$projectName    = "<PROJECT-NAME>"
$vstsToken      = "<YOUR-VSTS-TOKEN>"

$pools          = (Get-AzureDevOpsDeploymentGroups -projectName $projectName -organizationUri $devOpsURL -vstsToken $vstsToken)
$pool           = ($pools | Where-Object { $_.name -eq $poolName -or ($_.pool -and $_.pool.name -eq "$projectName-$poolName") } | Select-Object -First 1).pool

if (! $pool) {
    $pool = (Add-AzureDevOpsDeploymentGroup -name $poolName -projectName $projectName -organizationUri $devOpsURL -vstsToken $vstsToken).pool
}
```

## Install Local Deployment Agent

Download, Install, and Run a [Self Hosted Agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops) with PowerShell:

```PowerShell
# 1) replace placeholders in these variables:
$poolName       = "<POOL-NAME>"
$devOpsURL      = "https://dev.azure.com/<YOUR-ORGANIZATION>"
$projectName    = "<PROJECT-NAME>"
$vstsToken      = "<YOUR-VSTS-TOKEN>"
$tags           = @()    # List of Tags to identify the deployment agent
$credential     = $null  # or ... ([PSCredential]::new("username", (ConvertTo-SecureString -String "password" -AsPlainText -Force)))

# 2) Ensure, you download the newest Agent
$agentURL       = "https://vstsagentpackage.azureedge.net/agent/2.155.1/vsts-agent-win-x64-2.155.1.zip"

# 3) Prepare variables for execution
$agentFile      = "$($env:TEMP)/agent.zip"
$agentName      = $env:COMPUTERNAME

Write-Host "Download VSTS-Agent"
Invoke-WebRequest -Uri "$agentURL" -OutFile "$agentFile"

Write-Host "Extract VSTS-Agent"
Expand-Archive $agentFile "C:/agent"

$installCmd     = Get-AzureDevOpsAgentInstallParameters `
                        -deploymentGroup     $poolName `
                        -deploymentGroupTags $tags `
                        -projectName         $projectName `
                        -organizationUri     $devOpsURL `
                        -vstsToken           $vstsToken `
                        -agentName           $agentName `
                        -credential          $credential `
                        -runAsService:($credential -ne $null)

# Install the Deployment Agent
& cmd.exe /c """C:\agent\config.cmd $installCmd""" 2>%1
# Start the Deployment Agent
& cmd.exe /c """C:\agent\run.cmd"""
```

## Install Deployment Agent in NAV-/BC-Docker Container

To create the docker container use [NavContainerHelper](https://www.powershellgallery.com/packages/navcontainerhelper/) (see also [Freddy's Blog](https://freddysblog.com/category/navcontainerhelper/))

Download, Install, and Run a [Self Hosted Agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/v2-windows?view=azure-devops) as Service inside of a NAV-/BC-Docker Container with PowerShell:

```PowerShell
# 1) replace placeholders in these variables:
$containerName  = "<container-name>" # your BC container
$poolName       = "<POOL-NAME>"
$devOpsURL      = "https://dev.azure.com/<YOUR-ORGANIZATION>"
$projectName    = "<PROJECT-NAME>"
$vstsToken      = "<YOUR-VSTS-TOKEN>"
$tags           = @()    # List of Tags to identify the deployment agent
$vstsToken      = "<YOUR-VSTS-TOKEN>"
$credential     = ([PSCredential]::new("<docker-username>", (ConvertTo-SecureString -String "<docker-password>" -AsPlainText -Force)))

# 2) Ensure, you download the newest Agent
$agentURL       = "https://vstsagentpackage.azureedge.net/agent/2.155.1/vsts-agent-win-x64-2.155.1.zip"

# 3) Prepare variables for execution
$agentName      = $containerName
$installCmd     = Get-AzureDevOpsAgentInstallParameters `
                        -deploymentGroup     $poolName `
                        -deploymentGroupTags $tags `
                        -projectName         $projectName `
                        -organizationUri     $devOpsURL `
                        -vstsToken           $vstsToken `
                        -agentName           $agentName `
                        -credential          $credential `
                        -runAsService:($credential -ne $null)
$uninstallCmd   = Get-AzureDevOpsAgentUnInstallParameters -vstsToken $vstsToken

# Run this script inside of the Container
Invoke-ScriptInNavContainer -containerName $containerName -scriptblock {
        Param($agentURL, $installCmd, $uninstallCmd, $poolName, $agentName)

    try {
            Add-LocalGroupMember -Group Administrators -Member "Network Service" -ErrorAction SilentlyContinue

            Write-Host "Setup VSTS-Agent $agentName for pool $poolName" -f Cyan
            $agentFile = "C:/run/my/agent.zip"
            Set-Location "C:/run/my"

            Write-Host "Download VSTS-Agent"
            Invoke-WebRequest -Uri "$agentURL" -OutFile "$agentFile"

            Write-Host "Extract VSTS-Agent"
            Expand-Archive $agentFile "C:/agent"

            Write-Host "Register VSTS-Agent $agentName @ $poolName"

            # remove agent, if already registered
            & cmd.exe /c """C:\agent\config.cmd $uninstallCmd""" 2>%1

            # register agent
            & cmd.exe /c """C:\agent\config.cmd $installCmd""" 2>%1

            Write-Host "Setup VSTS-Agent done." -f Green
    } catch {
        Write-Warning "Install VSTS-Agent ERROR: $($_.Exception.Message)"
    }

} -argumentList $agentURL, $installCmd, $uninstallCmd, $poolName, $agentName

```
