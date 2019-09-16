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

# 3) Add / Update BC-DockerContainer parameter
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

$jobs = @()
foreach ($stage in $stages) {
    $containerParameters = $commonParameters
    $poolName            = $devOpsPools[$stage]
    $containerName       = $stage
    $licenseFile         = $licenseFiles[$stage]
    $agentName           = "docker-$poolName"

    Write-Host "Setup stage $stage [Pool: $($poolName) | Agent: $agentName | Container: $containerName]" -f Yellow

    $isDeploymentPool = $DeploymentPools.Contains($stage)

    $jobs += @(Start-Job { 
        Write-Host "Setup Container $containerName $isDeploymentPool" 
    } -Name "Setup stage $stage [Pool: $($poolName) | Agent: $agentName | Container: $containerName]" )

    New-NavContainer -containerName $containerName @containerParameters
}

$jobs | ForEach-Object { $_ | Wait-Job | Out-Null; Write-Host "$($_.Name) finished" }
Write-Host "All Done!"
```
