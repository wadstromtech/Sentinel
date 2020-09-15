# Recorded Future threat indicators ingestion
These templates speed up the onboarding process when ingesting Recorded Future threat intelligence into Azure Sentinel.

All templates are provided as-is without warranty.

## Deploy all templates
Before deploying the templates you need to create an app registration in Azure AD, the application only need the Microsoft Graph permission 'ThreatIndicators.ReadWrite.OwnedBy'.

See https://docs.microsoft.com/en-us/azure/active-directory/develop/quickstart-register-app for more information.

I'm not using the Logic App connector for Microsoft Graph Security because it requires both a highly privileged enterprise application, and a service account with at least Security Administrator RBAC permissions, which is way more than what we need in this case.

Once you have the applications clientId, clientSecret, the tenantId and your Recorded Future API key, the templates can be deployed with the following Powershell script (update the variables as necessary):
```Powershell
Import-Module -Name Az
Connect-AzAccount

# If you have multiple subscriptions, select the correct one with the below 2 lines
#Get-AzSubscription
#Select-AzSubscription '<subscriptionId>'

$gitPath = '<local path to git repo>'
Set-Location $gitPath
$batchingTemplates = Get-ChildItem '.\*.json' -Recurse | Where-Object {$_.FullName -like "*Batching*"}
$alertingTemplates = Get-ChildItem '.\*.json' -Recurse | Where-Object {$_.FullName -like "*Alerting*"}

$resourceGroup = '<resourceGroupName>'

$batchingParams = @{
    'app:tenantId'     = '<azureTenantId>';
    'app:clientId'     = '<clientId>';
    'app:clientSecret' = ConvertTo-SecureString -String '<clientSecret>' -AsPlainText -Force
}

foreach($batchingTemplate in $batchingTemplates)
{
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroup -TemplateFile $batchingTemplate.FullName @batchingParams -Verbose
}

$alertingParams = @{
    tenantId = '<azureTenantId>';
    recordedFutureApiKey = ConvertTo-SecureString -String '<recordedFutureApiKey>' -AsPlainText -Force
}

foreach($alertingTemplate in $alertingTemplates)
{
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroup -TemplateFile $alertingTemplate.FullName @alertingParams -Verbose
}
```

## Deploy an individual template
Each template can be deployed by pressing the 'Deploy to Azure' button.

**Note** you need to deploy the batching template before the alerting template.
