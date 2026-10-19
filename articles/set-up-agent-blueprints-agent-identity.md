# Agent 365: Blueprints and Agent Identities
<link rel="icon" href="articles//a365.svg" type="image/x-icon" >

## Blueprints and Agent Identities

Agent identity blueprints are a key part of the identity foundation behind Agent 365. A blueprint provides a reusable foundation for creating agent identities and managing shared identity configuration. Each agent identity is associated with a blueprint, and a single blueprint can support multiple agent identities.

On supported Microsoft platforms, some of the identity and blueprint provisioning is handled for you. The exact setup depends on the service you're using, so we'll look at what happens automatically and what you still need to configure.

For agents hosted outside Microsoft, we'll walk through registration options using the Agent 365 CLI and Microsoft Graph PowerShell cmdlets, where supported. We'll also cover bulk registration and when it makes sense to use it.

The sections below will take us through these options step by step.

## Agent Users and Autonomous Scenarios

In the final section, we'll explore agent users and how they support scenarios where an agent works independently rather than on behalf of a signed in user. Where supported and authorized, an agent user can give an agent a user identity for interacting with Microsoft 365 services such as SharePoint and Teams. Creating that identity does not automatically grant access to those services.

You'll still need to integrate the appropriate Agent 365 and Microsoft 365 authentication and telemetry components into your agent's runtime. The setup depends on whether you need telemetry only, delegated access through an on behalf of (OBO) flow, or autonomous operations using a supported app-only or service to service (S2S) flow. Permissions, consent, and service support determine what the agent can actually do.

## Register Blueprints and Agents with the Agent 365 CLI

### Prerequisites

We'll use the Agent 365 CLI as the primary tool for this walkthrough. Before running the setup commands, confirm the following:

- **.NET:** Install .NET 8.0 or later. See the [Windows installation guide](https://learn.microsoft.com/en-us/dotnet/core/install/windows), or use the operating system links in the [Agent 365 CLI documentation](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/agent-365-cli).
- **Azure CLI:** Install the Azure CLI and sign in to the tenant where you want to register the agent. Confirm the target subscription if you're also deploying Azure resources.
- **Sample code:** This walkthrough uses a Python Agent Framework sample. Start with the [Agent 365 SDK samples](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/samples?tabs=python) and adapt the code to your use case.
- **Hosting and runtime:** Choose a supported hosting option, such as a hosted agent in Microsoft Foundry or an Azure Web App. The Python sample used here targets Python 3.11 and uses the Microsoft 365 Agents SDK as its communication layer. Check the sample's dependencies before choosing a different Python version. For Node.js or .NET, check the requirements for the corresponding SDK and sample.
- **Permissions:** Azure resource deployment and agent identity provisioning require different permissions. Check whether Azure Contributor and Agent ID Developer are appropriate for the steps you're performing, and confirm any additional administrator or consent requirements in the [developer prerequisites](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/get-started#prerequisites).

### Install and Verify the CLI

Install the Agent 365 CLI globally:

```powershell
dotnet tool install --global Microsoft.Agents.A365.DevTools.Cli
```

Check the installed version:

```powershell
a365 --version
```

Commands and prompts can differ between releases. Use a current version and check `a365 setup all --help` before following the examples below. If you already have the CLI installed, update it with:

```powershell
dotnet tool update --global Microsoft.Agents.A365.DevTools.Cli
```

### Prepare Your Working Directory and Sign In

Create a dedicated folder for the agent and change to that directory before running setup. Keeping generated configuration, manifest, and tooling files together makes them easier to manage.

Sign in to the intended Azure tenant:

```powershell
az login --tenant "<tenant-id>"
```

The Agent 365 CLI may also prompt you to authenticate. Check that you're using the intended tenant and account for both tools.

### Register an Agent for S2S Scenarios

The following example starts the setup workflow for the agent. Confirm the authentication mode and requested permissions in your installed CLI version. To send telemetry back to Agent 365, you'll also need the appropriate OTel instrumentation and export configuration in your agent code; registration alone does not emit telemetry.

```powershell
a365 setup all --agent-name "agent-name-here" --verbose
```

The `--verbose` option displays additional diagnostic output. Leave it out if you don't need that detail, and review logs for sensitive information before sharing them.

During first-time setup, the CLI may need an enterprise application in your tenant. Current versions use the Microsoft-managed **Agent 365 CLI** application when available and can fall back to a tenant-owned application. If you see **"App Agent 365 CLI was not found"**, follow the [current CLI setup guidance](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/agent-365-cli#microsoft-entra-authentication) rather than assuming every version uses the same creation prompt.

Review the requested permissions before granting consent. If administrator consent is required, involve an authorized administrator and follow your organization's approval process. The screenshot below shows a prompt from the version used for this walkthrough; newer versions may look different.

<img width="327" height="70" alt="image" src="https://github.com/user-attachments/assets/57037312-0d4d-45f3-b820-b5978eaaba85" />


### Register an Agent for OBO and Microsoft 365 Integration

For an agent that acts on behalf of a signed-in user, the following example specifies OBO authentication and adds the `--m365` option for Microsoft 365 integration. Teams and Microsoft 365 Copilot provide the user-facing experience; your agent's runtime remains hosted in your chosen supported environment.

```powershell
a365 setup all --agent-name "agent-name-here" --authmode obo --m365
```

For a remotely hosted bot, provide the public HTTPS messaging endpoint that routes requests to your agent. Teams does not host the runtime itself. Confirm the endpoint and configuration requirements for your CLI version.

The workflow used here also requires preparing an app manifest and packaging it with the required icons. Upload the app package through the applicable Microsoft 365 admin center publishing workflow, then complete any required approvals and availability settings before other users can access it.

The following manifest is a sanitized example, not a deployment-ready configuration. Replace the placeholder IDs, URLs, and hostname with your own values. Generate the top-level manifest ID once for a new app; copy bot and authentication IDs from the corresponding app registrations rather than generating unrelated values.

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/teams/v1.26/MicrosoftTeams.schema.json",
  "manifestVersion": "1.26",
  "version": "1.0.1",
  "id": "00000000-0000-4000-8000-000000000001",
  "developer": {
    "name": "Your Organization",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "name": {
    "short": "A365 Agent Remote Runtime",
    "full": "A365 Agent Remote Runtime"
  },
  "description": {
    "short": "An Agent 365 sample for connecting a remotely hosted agent.",
    "full": "A365 Agent Remote Runtime is a sample configuration for connecting a remotely hosted custom engine agent to Microsoft Teams and Microsoft 365 Copilot. Replace the sample identifiers, domains, and developer URLs before packaging the app. Update this description to reflect your agent's actual capabilities, authentication requirements, data access, and retention practices."
  },
  "icons": {
    "color": "color.png",
    "outline": "outline.png"
  },
  "accentColor": "#155EEF",
  "bots": [
    {
      "botId": "00000000-0000-4000-8000-000000000002",
      "scopes": ["personal", "copilot"],
      "isNotificationOnly": false,
      "supportsFiles": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "commandLists": [
        {
          "scopes": ["personal", "copilot"],
          "commands": [
            {
              "title": "What can you do?",
              "description": "Explain your capabilities and limitations."
            },
            {
              "title": "Help me draft a message",
              "description": "Get help writing or improving a message."
            }
          ]
        }
      ]
    }
  ],
  "copilotAgents": {
    "customEngineAgents": [
      {
        "id": "00000000-0000-4000-8000-000000000002",
        "type": "bot",
        "disclaimer": {
          "text": "Sample AI agent. Responses may be inaccurate; verify important information. Do not submit sensitive data during testing."
        }
      }
    ]
  },
  "webApplicationInfo": {
    "id": "00000000-0000-4000-8000-000000000002",
    "resource": "api://botid-00000000-0000-4000-8000-000000000002"
  },
  "permissions": ["identity"],
  "validDomains": [
    "token.botframework.com",
    "example.com"
  ],
  "defaultInstallScope": "personal"
}
```

## Register Blueprints and Agents with PowerShell

If you prefer to work directly with APIs rather than the Agent 365 CLI, you can use the supported Microsoft Graph PowerShell cmdlets. The PowerShell walkthrough is still being prepared; commands, required permissions, and placeholder values will be added here.
The below command will create your blueprint and agent as well as provide the agent with the otel permission to send data back to the a365 control plane. 


```
#Requires -Version 7.0

$ErrorActionPreference = "Stop"

# ============================================================
# Configuration — replace placeholders before running
# ============================================================

$tenantId      = "Tenant_id_here"
$blueprintName = "AgentBlueprint"
$agentName     = "agentname"

$sponsorUserId = "user_object_id_here"
$ownerUserId   = "user_object_id_here"

# Optional. An agent user is not required for S2S OTEL export.
$createAgentUser = $false

$agentUserName     = "My Agent User"
$agentUserUpn      = "my-agent-user@contoso.onmicrosoft.com"
$agentUserNickname = "myagentuser"

# OTEL resource and application permission from your configuration should be the same.
$otelResourceAppId = "9b975845-388f-4429-889e-eab1ef63949c"
$otelWriteRoleId   = "8f71190c-00c8-461d-a63b-f74abde9ba52"

# Use federation/certificates in production if possible.
$secretLifetimeDays = 90

# ============================================================
# Helpers
# ============================================================

function Grant-ApplicationRole {
    param(
        [Parameter(Mandatory)]
        [string] $PrincipalObjectId,

        [Parameter(Mandatory)]
        [string] $ResourceObjectId,

        [Parameter(Mandatory)]
        [string] $RoleId
    )

    $existing = @(
        Get-MgServicePrincipalAppRoleAssignment `
            -ServicePrincipalId $PrincipalObjectId `
            -All |
        Where-Object {
            $_.ResourceId -eq $ResourceObjectId -and
            $_.AppRoleId -eq $RoleId
        }
    )

    if ($existing.Count -gt 0) {
        Write-Host "Permission already assigned: $RoleId"
        return
    }

    $request = @{
        Method      = "POST"
        Uri         = "https://graph.microsoft.com/v1.0/servicePrincipals/$ResourceObjectId/appRoleAssignedTo"
        ContentType = "application/json"
        Body        = @{
            principalId = $PrincipalObjectId
            resourceId  = $ResourceObjectId
            appRoleId   = $RoleId
        } | ConvertTo-Json
    }

    Invoke-MgGraphRequest @request | Out-Null
    Write-Host "Assigned permission: $RoleId"
}

function Get-BlueprintGraphHeaders {
    param(
        [Parameter(Mandatory)]
        [string] $Tenant,

        [Parameter(Mandatory)]
        [string] $ClientId,

        [Parameter(Mandatory)]
        [string] $ClientSecret
    )

    $request = @{
        Method      = "POST"
        Uri         = "https://login.microsoftonline.com/$Tenant/oauth2/v2.0/token"
        ContentType = "application/x-www-form-urlencoded"
        Body        = @{
            client_id     = $ClientId
            client_secret = $ClientSecret
            scope         = "https://graph.microsoft.com/.default"
            grant_type    = "client_credentials"
        }
    }

    $token = Invoke-RestMethod @request

    return @{
        Authorization   = "Bearer $($token.access_token)"
        "OData-Version" = "4.0"
    }
}

# ============================================================
# 1. Validate configuration and load modules
# ============================================================

foreach ($value in @($tenantId, $sponsorUserId, $ownerUserId)) {
    if ($value -match "<|>") {
        throw "Replace the tenant, sponsor, and owner placeholders first."
    }

    # These settings must be GUIDs, not UPNs.
    [void][guid]::Parse($value)
}

$modules = @(
    "Microsoft.Graph.Authentication"
    "Microsoft.Graph.Applications"
)

foreach ($module in $modules) {
    if (-not (Get-Module -ListAvailable -Name $module)) {
        Install-Module $module -Scope CurrentUser -Repository PSGallery
    }

    Import-Module $module
}

# ============================================================
# 2. Connect as the setup administrator
# ============================================================

$connectParams = @{
    TenantId     = $tenantId
    ContextScope = "Process"
    UseDeviceCode = $true
    ErrorAction  = "Stop"
    Scopes       = @(
        "AgentIdentityBlueprint.Create"
        "AgentIdentityBlueprintPrincipal.Create"
        "AgentIdentityBlueprint.AddRemoveCreds.All"
        "Application.Read.All"
        "AppRoleAssignment.ReadWrite.All"
    )
}

Connect-MgGraph @connectParams

# Validate the OTEL resource/role before creating new objects.
$otelResources = @(
    Get-MgServicePrincipal `
        -Filter "appId eq '$otelResourceAppId'" `
        -Property Id, AppId, DisplayName, AppRoles
)

if ($otelResources.Count -ne 1) {
    throw "Expected exactly one OTEL resource service principal in this tenant."
}

$otelResource = $otelResources[0]

$otelRoles = @(
    $otelResource.AppRoles | Where-Object {
        $_.Id -eq $otelWriteRoleId -and
        $_.IsEnabled -and
        $_.AllowedMemberTypes -contains "Application"
    }
)

if ($otelRoles.Count -ne 1) {
    throw "The configured OTEL role is not an enabled application permission."
}

Write-Host "OTEL resource: $($otelResource.DisplayName)"
Write-Host "OTEL permission: $($otelRoles[0].Value)"

# Resolve the optional agent-user provisioning permission.
if ($createAgentUser) {
    $graphResources = @(
        Get-MgServicePrincipal `
            -Filter "appId eq '00000003-0000-0000-c000-000000000000'" `
            -Property Id, AppRoles
    )

    if ($graphResources.Count -ne 1) {
        throw "Microsoft Graph service principal was not found."
    }

    $graphResource = $graphResources[0]

    $agentUserRoles = @(
        $graphResource.AppRoles | Where-Object {
            $_.Value -eq "AgentIdUser.ReadWrite.IdentityParentedBy" -and
            $_.IsEnabled -and
            $_.AllowedMemberTypes -contains "Application"
        }
    )

    if ($agentUserRoles.Count -ne 1) {
        throw "AgentIdUser.ReadWrite.IdentityParentedBy was not found."
    }
}

# ============================================================
# 3. Create the blueprint
# ============================================================

$request = @{
    Method      = "POST"
    Uri         = "https://graph.microsoft.com/v1.0/applications/microsoft.graph.agentIdentityBlueprint"
    Headers     = @{ "OData-Version" = "4.0" }
    ContentType = "application/json"
    Body        = @{
        "@odata.type"        = "#microsoft.graph.agentIdentityBlueprint"
        displayName          = $blueprintName
        "sponsors@odata.bind" = @(
            "https://graph.microsoft.com/v1.0/users/$sponsorUserId"
        )
        "owners@odata.bind" = @(
            "https://graph.microsoft.com/v1.0/users/$ownerUserId"
        )
    } | ConvertTo-Json -Depth 5
}

$blueprint = Invoke-MgGraphRequest @request

$blueprintObjectId = $blueprint.id
$blueprintAppId    = $blueprint.appId

if (-not $blueprintObjectId -or -not $blueprintAppId) {
    throw "Blueprint response is missing id or appId."
}

Write-Host "Blueprint object ID: $blueprintObjectId"
Write-Host "Blueprint client ID: $blueprintAppId"

# ============================================================
# 4. Create the blueprint principal
# ============================================================

$request = @{
    Method      = "POST"
    Uri         = "https://graph.microsoft.com/v1.0/servicePrincipals/microsoft.graph.agentIdentityBlueprintPrincipal"
    Headers     = @{ "OData-Version" = "4.0" }
    ContentType = "application/json"
    Body        = @{ appId = $blueprintAppId } | ConvertTo-Json
}

$blueprintPrincipal = Invoke-MgGraphRequest @request
$blueprintSpId = $blueprintPrincipal.id

if (-not $blueprintSpId) {
    throw "Blueprint principal response is missing its object ID."
}

Write-Host "Blueprint principal object ID: $blueprintSpId"

# ============================================================
# 5. Create and validate a local-development secret
# ============================================================

# Use the blueprint created in section 3.
# Do not insert a hardcoded recovery client ID here.
$blueprintObjectId = $blueprint.id
$blueprintAppId    = $blueprint.appId

if (-not $blueprintObjectId -or -not $blueprintAppId) {
    throw "Section 3 did not return the blueprint object ID and client ID."
}

# Confirm the current Graph session belongs to the intended tenant.
$context = Get-MgContext

if (-not $context -or $context.TenantId -ne $tenantId) {
    throw "The administrator Graph session is not connected to the configured tenant."
}

# Confirm the application object and client ID belong together.
$blueprintApplication = Get-MgApplication `
    -ApplicationId $blueprintObjectId `
    -Property Id, AppId, DisplayName `
    -ErrorAction Stop

if ($blueprintApplication.AppId -ne $blueprintAppId) {
    throw "Blueprint object ID and client ID do not match."
}

Write-Host "Creating credential for: $($blueprintApplication.DisplayName)"
Write-Host "Blueprint object ID: $blueprintObjectId"
Write-Host "Blueprint client ID: $blueprintAppId"

$credentialParameters = @{
    ApplicationId = $blueprintObjectId
    PasswordCredential = @{
        displayName = "Local development"
        endDateTime = (Get-Date).ToUniversalTime().AddDays(
            $secretLifetimeDays
        )
    }
    ErrorAction = "Stop"
}

$secretResponse = Add-MgApplicationPassword @credentialParameters

if ([string]::IsNullOrWhiteSpace($secretResponse.SecretText)) {
    throw "Secret creation returned no secret value. Stopping."
}

# Use the returned VALUE, never KeyId.
$blueprintSecret = $secretResponse.SecretText

$tokenRequest = @{
    Method      = "POST"
    Uri         = "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token"
    ContentType = "application/x-www-form-urlencoded"
    Body        = @{
        client_id     = $blueprintAppId
        client_secret = $blueprintSecret
        scope         = "https://graph.microsoft.com/.default"
        grant_type    = "client_credentials"
    }
    ErrorAction = "Stop"
}

# Credential validation runs INSIDE the script while variables exist.
$clientIdsMatch = (
    $tokenRequest.Body.client_id -eq $blueprintApplication.AppId
)

$secretValuesMatch = (
    $tokenRequest.Body.client_secret -ceq $secretResponse.SecretText
)

$usesSecretId = (
    [string]$tokenRequest.Body.client_secret -ceq
    [string]$secretResponse.KeyId
)

[pscustomobject]@{
    GraphTenantId           = $context.TenantId
    BlueprintName           = $blueprintApplication.DisplayName
    BlueprintClientId       = $blueprintAppId
    CredentialId            = $secretResponse.KeyId
    CredentialStart         = $secretResponse.StartDateTime
    CredentialExpiry        = $secretResponse.EndDateTime
    ClientIdsMatch          = $clientIdsMatch
    UsesReturnedSecretValue = $secretValuesMatch
    UsesSecretIdByMistake   = $usesSecretId
} | Format-List

if (-not $clientIdsMatch -or -not $secretValuesMatch -or $usesSecretId) {
    throw "Credential validation failed. Token request was not sent."
}

# Display once, as requested. This will appear in terminal scrollback.
Write-Host "Client secret VALUE: $blueprintSecret"
Write-Warning "Store this value securely; Entra cannot return it again."

# Verify authentication before continuing.
$tokenResponse = $null

for ($attempt = 1; $attempt -le 13; $attempt++) {
    try {
        $tokenResponse = Invoke-RestMethod @tokenRequest
        break
    } catch {
        $tokenError = $null

        try {
            $tokenError = $_.ErrorDetails.Message |
                ConvertFrom-Json -ErrorAction Stop
        } catch {
            # Leave tokenError empty if the response isn't JSON.
        }

        $isInvalidSecret = (
            $tokenError -and
            $tokenError.error_codes -contains 7000215
        )

        # Only allow a short retry window immediately after secret creation.
        if (-not $isInvalidSecret -or $attempt -eq 13) {
            throw
        }

        Write-Host (
            "New credential not accepted yet; retrying in 10 seconds " +
            "(attempt $attempt of 13)."
        )

        Start-Sleep -Seconds 10
    }
}

if ([string]::IsNullOrWhiteSpace($tokenResponse.access_token)) {
    throw "Authentication returned no access token."
}

$headers = @{
    Authorization   = "Bearer $($tokenResponse.access_token)"
    "OData-Version" = "4.0"
}

Write-Host "Blueprint authentication succeeded."

# ============================================================
# 6. Optional: authorize the blueprint to create agent users
# ============================================================

if ($createAgentUser) {
    Grant-ApplicationRole `
        -PrincipalObjectId $blueprintSpId `
        -ResourceObjectId $graphResource.Id `
        -RoleId $agentUserRoles[0].Id
}

# ============================================================
# 7. Authenticate as the blueprint and create the agent
# ============================================================

# Reuse the blueprint Graph token obtained and tested in section 5.
if (
    -not $tokenResponse -or
    [string]::IsNullOrWhiteSpace($tokenResponse.access_token)
) {
    throw "Section 5 did not obtain a blueprint token. Stop before creating the agent."
}

if ($tokenRequest.Body.client_id -ne $blueprintAppId) {
    throw "The blueprint client ID changed after section 5."
}

$headers = @{
    Authorization   = "Bearer $($tokenResponse.access_token)"
    "OData-Version" = "4.0"
}

$request = @{
    Method      = "POST"
    Uri         = "https://graph.microsoft.com/v1.0/servicePrincipals/microsoft.graph.agentIdentity"
    Headers     = $headers
    ContentType = "application/json"
    Body        = @{
        displayName              = $agentName
        agentIdentityBlueprintId = $blueprintAppId
        "sponsors@odata.bind" = @(
            "https://graph.microsoft.com/v1.0/users/$sponsorUserId"
        )
    } | ConvertTo-Json -Depth 5
}

$agentIdentity = Invoke-RestMethod @request
$agentIdentityId = $agentIdentity.id

if (-not $agentIdentityId) {
    throw "Agent creation response is missing its object ID."
}

# Read appId explicitly because create responses can omit it.
# Allow a newly created agent time to propagate.
$retryTimeoutSeconds  = 120
$retryIntervalSeconds = 10
$retryTimer = [System.Diagnostics.Stopwatch]::StartNew()
$agentServicePrincipal = $null

while ($true) {
    try {
        $agentServicePrincipal = Get-MgServicePrincipal `
            -ServicePrincipalId $agentIdentityId `
            -Property Id, AppId, DisplayName `
            -ErrorAction Stop

        if (-not $agentServicePrincipal.AppId) {
            throw "Agent was found, but its appId was not returned."
        }

        break
    } catch {
        # Retry only resource-not-found errors.
        # Permission, authentication, and other failures stop immediately.
        $errorText = @(
            $_.Exception.Message
            $_.ErrorDetails.Message
            $_.FullyQualifiedErrorId
        ) -join " "

        $isNotFound = $errorText -match (
            "Request_ResourceNotFound|\b404\b"
        )

        $remainingSeconds = $retryTimeoutSeconds -
            $retryTimer.Elapsed.TotalSeconds

        if (-not $isNotFound -or $remainingSeconds -le 0) {
            throw
        }

        $waitSeconds = [Math]::Min(
            $retryIntervalSeconds,
            $remainingSeconds
        )

        Write-Host (
            "Agent not visible yet. Retrying in {0:N0}s ({1:N0}s remaining)..." -f
            $waitSeconds, $remainingSeconds
        )

        Start-Sleep -Milliseconds ([int]($waitSeconds * 1000))
    }
}

$retryTimer.Stop()
$agentIdentityAppId = $agentServicePrincipal.AppId

Write-Host "Agent object ID: $agentIdentityId"
Write-Host "Agent client ID: $agentIdentityAppId"

# ============================================================
# 8. Grant OTEL application permission directly to the agent
# ============================================================

# This uses the administrator's Graph session established in step 2.
Grant-ApplicationRole `
    -PrincipalObjectId $agentIdentityId `
    -ResourceObjectId $otelResource.Id `
    -RoleId $otelWriteRoleId

Write-Host "OTEL application permission is assigned to the agent identity."

# ============================================================
# 9. Optional: create the agent user
# ============================================================

$agentUser = $null

if ($createAgentUser) {
    # Refresh the token after granting the provisioning permission.
    $headers = Get-BlueprintGraphHeaders `
        -Tenant $tenantId `
        -ClientId $blueprintAppId `
        -ClientSecret $blueprintSecret

    $request = @{
        Method      = "POST"
        Uri         = "https://graph.microsoft.com/v1.0/users/microsoft.graph.agentUser"
        Headers     = $headers
        ContentType = "application/json"
        Body        = @{
            accountEnabled    = $true
            displayName       = $agentUserName
            userPrincipalName = $agentUserUpn
            mailNickname      = $agentUserNickname
            identityParentId  = $agentIdentityId
        } | ConvertTo-Json
    }

    for ($attempt = 1; $attempt -le 10; $attempt++) {
        try {
            $agentUser = Invoke-RestMethod @request
            Write-Host "Created agent user: $($agentUser.id)"
            break
        } catch {
            $details = $_.ErrorDetails.Message

            # Retry only the specific parent-propagation error.
            if (
                $details -match "IdentityParent does not exist" -and
                $attempt -lt 10
            ) {
                Write-Host "Waiting 10 seconds for agent-parent propagation..."
                Start-Sleep -Seconds 10
                continue
            }

            # Permission, conflict, and other errors remain visible.
            throw
        }
    }
}

# ============================================================
# 10. Display non-secret configuration
# ============================================================

[pscustomobject]@{
    TenantId                  = $tenantId
    BlueprintObjectId         = $blueprintObjectId
    BlueprintClientId         = $blueprintAppId
    BlueprintPrincipalId      = $blueprintSpId
    AgentObjectId             = $agentIdentityId
    AgentClientId             = $agentIdentityAppId
    AgentUserObjectId          = $(if ($agentUser) { $agentUser.id })
    AgentUserUPN               = $(if ($agentUser) { $agentUser.userPrincipalName })
    OtelResourceClientId      = $otelResourceAppId
    OtelPermission            = $otelRoles[0].Value
    DevelopmentSecretExpires = $secretResponse.EndDateTime
} | Format-List

Disconnect-MgGraph | Out-Null
```














































## References

[Install and use the Agent 365 CLI](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/agent-365-cli?tabs=windows)
[Agent 365 CLI](https://learn.microsoft.com/en-us/microsoft-agent-365/developer/reference/cli/setup)
