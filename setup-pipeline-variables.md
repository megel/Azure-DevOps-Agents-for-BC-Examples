# Setup Pipeline-Variables

## Prerequisites

1) Get your [VSTS-Token](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops)
1) Install [AzureDevOpsAPIUtils](https://www.powershellgallery.com/packages/AzureDevOpsAPIUtils) with PowerShell:

```PowerShell
Install-Module AzureDevOpsAPIUtils -Force
Update-Module  AzureDevOpsAPIUtils
Import-Module  AzureDevOpsAPIUtils -Force
```

## Create an Azure DevOps Variable Group

Create an Azure DevOps [Variable Group](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml) for your organization with PowerShell:

```PowerShell
# 1) replace placeholders in these variables:
$groupName      = "<VARIABLE-GROUP-NAME>"
$description    = "<DESCRIPTION-FOR-THE-GROUP>"
$devOpsURL      = "https://dev.azure.com/<YOUR-ORGANIZATION>"
$projectName    = "<PROJECT-NAME>"
$vstsToken      = "<YOUR-VSTS-TOKEN>"

# 2) add your VISIBLE variables:
$variables      = @{
    "DockerImage"      = "mcr.microsoft.com/businesscentral/onprem:w1";
    "VsixDownloadPath" = "C:/run/*.vsix";
}
# 2) add your HIDDEN variables:
$secrets      = @{
    "LicenseFile"      = "";
}

# 3) Create the Variable Group
Add-AzureDevOpsVariableGroup `
    -name            $groupName `
    -description     $description `
    -projectName     $projectName `
    -organizationUri $devOpsURL `
    -vstsToken       $vstsToken `
    -variables       $variables `
    -secrets         $secrets
```

## Update an Azure DevOps Variable Group

Update an Azure DevOps [Variable Group](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&tabs=yaml) for your organization with PowerShell:

***NOTE:*** This may **replace existing variables** at the specific variable group

```PowerShell
# 1) replace placeholders in these variables:
$groupName      = "<VARIABLE-GROUP-NAME>"
$description    = "<DESCRIPTION-FOR-THE-GROUP>"
$devOpsURL      = "https://dev.azure.com/<YOUR-ORGANIZATION>"
$projectName    = "<PROJECT-NAME>"
$vstsToken      = "<YOUR-VSTS-TOKEN>"

# 2) add your VISIBLE variables:
$variables      = @{
    "DockerImage"      = "mcr.microsoft.com/businesscentral/onprem:w1";
    "VsixDownloadPath" = "C:/run/*.vsix";
}
# 2) add your HIDDEN variables:
$secrets      = @{
    "LicenseFile"      = "";
}

# 3) Get the right Variable Group
$groups = Get-AzureDevOpsVariableGroups -projectName $projectName -organizationUri $devOpsURL -vstsToken $vstsToken
$group  = ($groups | Where-Object { $_.name -eq $groupName } | Select-Object -First 1)

# 4) Update the Variable Group
#    NOTE: Existing variables May be removed!!!
Update-AzureDevOpsVariableGroup `
    -groupId         $group.id `
    -name            $groupName `
    -description     $description `
    -projectName     $projectName `
    -organizationUri $devOpsURL `
    -vstsToken       $vstsToken `
    -variables       $variables `
    -secrets         $secrets
```
