# TestManagement Tool Azure Operations

This repository is the operational reference for the MLB TestPlan toolchain running in Azure. It centralizes the current deployment inventory, Docker container mapping, Azure Container Apps details, Cloud Shell deployment commands, verification steps, and access URLs required to operate the environment safely.

The application source code lives in the MLB TestPlan application repository. This repository is intentionally documentation-first so future maintenance does not depend on searching chat history or old shell sessions.

## Scope

This operations repository documents:

1. The current Azure resource layout.
2. The three deployed containerized services.
3. The Docker-to-Container-App mapping.
4. The Cloud Shell deployment workflow.
5. The access URLs for current production endpoints.
6. The commands used to verify revisions, images, traffic, and health.

## Current Environment Summary

| Item | Value |
|---|---|
| Application code branch | `pre-final` |
| Azure resource group | `mlb-testplan-mj-rg` |
| Azure Container Registry | `mjmlbtestplanacr2026` |
| Azure region | `germanywestcentral` |
| Cloud Shell working copy | `~/mlb-testplan-mcp_latest` |

## Current Azure Services

| Service | Azure Container App | Purpose |
|---|---|---|
| Main dashboard/API | `mlb-testplan` | Serves the dashboard and root API |
| Jira/Xray proxy | `jira-mcp` | Serves Jira and Xray-backed helper APIs |
| Confluence proxy | `confluence-mcp` | Serves Confluence-backed helper APIs |

## Current Access URLs

| Surface | URL |
|---|---|
| Root app | `https://mlb-testplan.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/` |
| Dashboard | `https://mlb-testplan.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/dashboard` |
| Jira MCP docs | `https://jira-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs` |
| Confluence MCP docs | `https://confluence-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs` |

## Document Index

| Document | Purpose |
|---|---|
| [docs/AZURE_INFRASTRUCTURE.md](docs/AZURE_INFRASTRUCTURE.md) | Resource group, registry, container apps, URL inventory, and service mapping |
| [docs/DOCKER_CONTAINERS.md](docs/DOCKER_CONTAINERS.md) | Local Docker service inventory, container names, environment wiring, and container commands |
| [docs/DEPLOYMENT_RUNBOOK.md](docs/DEPLOYMENT_RUNBOOK.md) | End-to-end deployment workflow for current and future changes |
| [docs/ACCESS_AND_OPERATIONS.md](docs/ACCESS_AND_OPERATIONS.md) | Day-2 operations, verification, logs, revisions, troubleshooting, and access guidance |

## Operating Principle

GitHub push alone does not deploy to Azure.

Every real deployment requires:

1. Updating code in the application repository.
2. Pushing the branch.
3. Pulling the latest branch in Azure Cloud Shell.
4. Building a new image tag in ACR.
5. Updating the matching Azure Container App to that new image.
6. Verifying the deployed revision and public URL.

## Change Management Notes

1. Always use a new image tag for every deployment.
2. Do not assume the latest Git commit is already running in Azure.
3. Validate the exact deployed image after every update.
4. If the UI still appears stale after deployment, inspect the active revision and then rule out browser cache.

## Recommended Future Use

Use this repository as the source of truth for:

1. New team-member onboarding to the Azure environment.
2. Routine deployments from Cloud Shell.
3. Incident response when a service appears stale or unhealthy.
4. Auditing how each deployed container maps back to the application structure.