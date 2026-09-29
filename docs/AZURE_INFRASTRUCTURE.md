# Azure Infrastructure Inventory

This document records the current Azure footprint used by the MLB TestPlan toolchain.

## Resource Inventory

| Resource Type | Name | Notes |
|---|---|---|
| Resource Group | `mlb-testplan-mj-rg` | Primary resource group for the deployed environment |
| Azure Container Registry | `mjmlbtestplanacr2026` | Stores the built images for all services |
| Container App | `mlb-testplan` | Main dashboard and API |
| Container App | `jira-mcp` | Jira and Xray proxy service |
| Container App | `confluence-mcp` | Confluence proxy service |
| Azure Region | `germanywestcentral` | Public endpoint suffix reflects this region |
| Git branch used for deployment | `pre-final` | Application branch currently associated with this environment |
| Cloud Shell repo path | `~/mlb-testplan-mcp_latest` | Expected Azure-side working copy |

## Public Endpoints

| Service | Endpoint |
|---|---|
| `mlb-testplan` root | `https://mlb-testplan.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/` |
| `mlb-testplan` dashboard | `https://mlb-testplan.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/dashboard` |
| `jira-mcp` docs | `https://jira-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs` |
| `confluence-mcp` docs | `https://confluence-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs` |

## Service Responsibilities

### `mlb-testplan`

- Serves the main dashboard.
- Hosts the root FastAPI endpoints.
- Depends functionally on the Jira MCP and Confluence MCP services.

### `jira-mcp`

- Exposes Jira-backed and Xray-backed helper APIs.
- Supports dashboard features that depend on Jira issues, Xray test plans, test executions, and test runs.

### `confluence-mcp`

- Exposes Confluence-backed helper APIs.
- Supports page retrieval and parsing workflows used by the root dashboard.

## Docker Container Mapping

The local Docker Compose layout maps directly to the Azure deployment model.

| Local Compose Service | Local Container Name | Azure Container App | Internal Role |
|---|---|---|---|
| `mlb-testplan` | `mlb-testplan-mcp` | `mlb-testplan` | Main dashboard and API |
| `jira-mcp` | `mlb-jira-mcp` | `jira-mcp` | Jira and Xray helper service |
| `confluence-mcp` | `mlb-confluence-mcp` | `confluence-mcp` | Confluence helper service |

## Local Compose Runtime Wiring

The main app depends on the helper services through these environment values:

```text
XRAY_BASE_URL=http://jira-mcp:8000
LOCAL_API_URL=http://confluence-mcp:8001
CONFLUENCE_PAGE_ID=2378907792
```

This matters operationally because Azure symptoms in the root dashboard may actually be caused by a stale or unhealthy helper service.

## ACR Image Naming Convention

| Service | Image Name Pattern |
|---|---|
| `mlb-testplan` | `mjmlbtestplanacr2026.azurecr.io/mlb-testplan:vNEXT` |
| `jira-mcp` | `mjmlbtestplanacr2026.azurecr.io/jira-mcp:vNEXT` |
| `confluence-mcp` | `mjmlbtestplanacr2026.azurecr.io/confluence-mcp:vNEXT` |

## Operational Invariants

1. A new GitHub commit does not automatically update the running Azure service.
2. A new ACR image tag must be built for the changed service.
3. The matching Azure Container App must be updated to use that new image.
4. Verification must be done against the deployed URL and the active revision, not against assumptions.

## Recommended Updates To This Document

Update this file whenever any of the following change:

1. Resource group name.
2. Container registry name.
3. Container App names.
4. Public URLs.
5. Cloud Shell working directory.
6. Deployment branch.
