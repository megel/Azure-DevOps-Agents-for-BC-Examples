# Create BC-DockerContainer for Staging Simulation

```PowerShell
$stages         = @(
    "build-nst"  # Build Agent
    "d-stage"    # Development
    "q-stage"    # Quality Assurance
    "u-stage"    # User Acceptance Test
    "p-stage"    # Production
    "unit-test"  # Test-Automation
)

# 1) Update Common Information used for BC-DockerContainer
$imageName       = "mcr.microsoft.com/businesscentral/onprem:w1"
$shortcuts       = "Desktop"
$devLicenseFile  = ""  # Add the URL to your DEV License File
$custLicenseFile = ""  # Add the URL to your CUSTOMER License File
# Credential for the container
$credential      = ([PSCredential]::new("username", (ConvertTo-SecureString -String "password" -AsPlainText -Force)));

# 2) Add/Update Stage-Specific information
$devOpsPools     = @{
    "build-nst" = "D365BC-NST";
    "d-stage"   = "d-stage";
    "q-stage"   = "q-stage";
    "u-stage"   = "u-stage";
    "p-stage"   = "p-stage";
    "unit-test" = "unit-tests";
}
$licenseFiles    = @{
    "build-nst" = $devLicenseFile;
    "d-stage"   = $devLicenseFile;
    "q-stage"   = $custLicenseFile;
    "u-stage"   = $custLicenseFile;
    "p-stage"   = $custLicenseFile;
    "unit-test" = $devLicenseFile;
}
$DeploymentPools = @("d-stage", "q-stage", "u-stage", "p-stage", "unit-test")

# 3) Ensure, the pool is created for the Stage
$stages | ForEach-Object {
    $poolName       = $devOpsPools[$_]

    if ($DeploymentPools.Contains($stage)) {
        # Deployment Agent Pool/Group
        $pools          = (Get-AzureDevOpsDeploymentGroups -projectName $projectName -organizationUri $devOpsURL -vstsToken $vstsToken)
        $pool           = ($pools | Where-Object { $_.name -eq $poolName -or ($_.pool -and $_.pool.name -eq "$projectName-$poolName") } | Select-Object -First 1).pool

        if (! $pool) {
            $pool = (Add-AzureDevOpsDeploymentGroup -name $poolName -projectName $projectName -organizationUri $devOpsURL -vstsToken $vstsToken).pool
        }
    } else {
        # Build Agent Pool
        $pools          = (Get-AzureDevOpsAgentPools -organizationUri $devOpsURL -vstsToken $vstsToken)
        $pool           = ($pools | Where-Object { $_.name -eq $poolName } | Select-Object -First 1)

        if (! $pool) {
            $pool = (Add-AzureDevOpsAgentPool -name $poolName -organizationUri $devOpsURL -vstsToken $vstsToken)
        }
    }
}

# 4) Add / Update BC-DockerContainer parameter
$commonParameters      = @{
    "accept_eula"              = $true;
    "accept_outdated"          = $true;
    "imageName"                = $imageName;
    "auth"                     = "NAVUserPassword";
    "Credential"               = $credential;
    "alwaysPull"               = $true;
    "useBestContainerOS"       = $true;
    "shortcuts"                = $shortcuts;
    "useSSL"                   = $true;
    "updateHosts"              = $true;
    "dns"                      = "8.8.8.8" # needed to Install & Run the Agent Insight of the Container
}


foreach ($stage in $stages) {
    $containerParameters = $commonParameters
    $poolName            = $devOpsPools[$stage]
    $containerName       = $stage
    $licenseFile         = $licenseFiles[$stage]
    $agentName           = "docker-$poolName"

    Write-Host "Setup stage $stage [Pool: $($poolName) | Agent: $agentName | Container: $containerName]" -f Yellow

    $isDeploymentPool = $DeploymentPools.Contains($stage)

    if ("$licenseFiles" -ne "") {
        $containerParameters += @{"licenseFile" = $licenseFile}
    }

    if ($isDeploymentPool) {
        $installCmd          = Get-AzureDevOpsAgentInstallParameters -deploymentGroup $poolName -projectName $projectName -organizationUri $devOpsURL -vstsToken $vstsToken -agentName $containerName -credential $credential -runAsService -deploymentGroupTags @($stage)
    } else {
        $installCmd          = Get-AzureDevOpsAgentInstallParameters -poolName $poolName -organizationUri $devOpsURL -vstsToken $vstsToken -agentName $containerName -credential $credential -runAsService
    }
    $uninstallCmd = Get-AzureDevOpsAgentUnInstallParameters -vstsToken $vstsToken

    if ($null -ne $licenseFile -and $licenseFile -ne "") {
        $containerParameters["licenseFile"] = $licenseFile
    }

    New-NavContainer -containerName $containerName @containerParameters

    Invoke-ScriptInNavContainer -containerName $containerName -scriptblock {
        Param($agentURL, $installCmd, $uninstallCmd, $poolName, $agentName)

        try {
                Write-Host "Setup VSTS-Agent $agentName for pool $poolName" -f Cyan
                $agentFile = "C:/run/my/agent.zip"
                Set-Location "C:/run/my"

                Write-Host "Download VSTS-Agent"
                Invoke-WebRequest -Uri "$agentURL" -OutFile "$agentFile"

                Write-Host "Extract VSTS-Agent"
                Expand-Archive $agentFile "C:/agent"

                Write-Host "Un-Register VSTS-Agent"
                & cmd.exe /c """C:\agent\config.cmd $uninstallCmd""" 2>%1
                Write-Host "Register VSTS-Agent"
                & cmd.exe /c """C:\agent\config.cmd $installCmd""" 2>%1

                Write-Host "Setup VSTS-Agent done." -f Green

                Add-LocalGroupMember -Group Administrators -Member "Network Service" -ErrorAction SilentlyContinue
        } catch {
            Write-Warning "Install VSTS-Agent ERROR: $($_.Exception.Message)"
        }

    } -argumentList $agentURL, $installCmd, $uninstallCmd, $poolName, $agentName

    Write-Host "Setup stage $stage [Pool: $($poolName) | Agent: $agentName | Container: $containerName] done." -f Green
}
```
