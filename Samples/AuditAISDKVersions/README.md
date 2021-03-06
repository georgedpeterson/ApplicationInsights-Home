# Audit the SDK versions used across your organization

## Why Audit?

Application Insights SDKs get updated periodically, fixing issues and introducing new capabilities every now and then. The portal experiences offer the most value if all application components are using SDK versions that introduced certain functionality, if not the latest stable SDKs. For instance, the [Application Map](https://docs.microsoft.com/azure/application-insights/app-insights-app-map) and [End-to-end transaction diagnostics](https://docs.microsoft.com/azure/application-insights/app-insights-transaction-diagnostics) experiences offer cross-component visibility. If any of the (say) .NET standard components were using an SDK version older than 2.4, then these triage and diagnostics experiences will be limited across such components.

Periodically auditing the application components, to find those needing SDK updates - helps make the most of Application Insights.

## Automate Audit

Your organization may have hundreds or thousands of application components spread across multiple teams. Tracking the SDK versions used across these components and ensuring that they are using newer than a given SDK version is hard. The ReportAISDKVersions powershell script here queries Application Insights resources in specified subscriptions to generate a report on what SDK versions they are using. You can then use this report to determine what components/teams need to update the SDK versions.

### ReportAISDKVersions
This script takes a comma separated string of subscription IDs or names:

``` .\ReportAISDKVersions.ps1 -subscriptions "subscription-name-1,subscription-id-2" ```

It then iterates through all AI resources the user has access to in the subscription(s), and performing the following for each of them: 
1. Create an API key for the audit report.
2. Query the AI resource with the new API key, for the SDK versions used by all roles reporting to that AI resource. It checks for **server side telemetry only**, requests in particular.
3. Parse the SDK versions based on [standard prefixes](https://github.com/Microsoft/ApplicationInsights-Home/blob/master/EndpointSpecs/SDK-VERSIONS.md#sdk-names).
4. Delete the API key created in step 1.
5. Add a row into the report for each role in this AI resource, with the SDK version found. If there were any errors, or the resource had no **request** data, it is noted in the last column.

The output is a file **AISDKVersionAuditReport-yyyy-MM-dd-hh-mm.csv** generated in the same directory as the script. See AISDKVersionAuditReport-2018-08-29-10-52.csv in this directory for an example.

**NOTE**: 
* The script can take roughly 2-5 seconds for each AI resource. Depending on how many AI resources you have in the subscriptions, you may want to group 2-5 subscriptions in each run.
* If an AI resource has exhausted the quota on API keys (10 per resource), then the script will fail for that AI resource. Delete any API keys you no longer need. 
* If the script fails to delete the API key for a resource (say the resource is locked to prevent any deletes), the failure is noted in the output file. Be sure to manually delete the key before the next execution. Otherwise the script will fail for that AI resource the next time - will not be able to create another API key with the same description.
* You can change the vaue for the $apiKeyDescription in the next run as a mitigation step.

### Requirements

Use the Connect-AzureRmAccount commandlet to authenticate with Azure. The ReportAISDKVersions powershell script uses the following ARM commandlets:
* [Set-AzureRmContext](https://docs.microsoft.com/powershell/module/azurerm.profile/set-azurermcontext?view=azurermps-6.8.0)
* [Get-AzureRmApplicationInsights](https://docs.microsoft.com/powershell/module/azurerm.applicationinsights/get-azurermapplicationinsights?view=azurermps-6.8.0)
* [New-AzureRmApplicationInsightsApiKey](https://docs.microsoft.com/powershell/module/azurerm.applicationinsights/get-azurermapplicationinsightsapikey?view=azurermps-6.8.0)
* [Remove-AzureRmApplicationInsightsApiKey](https://docs.microsoft.com/powershell/module/azurerm.applicationinsights/remove-azurermapplicationinsightsapikey?view=azurermps-6.8.0)

## Next steps

The script checks for SDK versions. You can customize this script to audit for the presence or absence of any attribute in the telemetry generated by your application. With Azure monitoring/Applicatio Insights, anything that can be queried for, can be alerted on, or audited on. Customize this script to audit for any custom processing of telemetry is consistently applied across all components for instance.

* See [dev.applicationinsights.io](https://dev.applicationinsights.io/) for more information on querying application insights
* See [docs.loganalytics.io](https://docs.loganalytics.io/index) to learn more about the query language