# Deployment Runbook

This runbook documents how to deploy current and future changes to the Azure-hosted MLB TestPlan environment.

## Core Rule

Pushing to GitHub is not a deployment.

A complete deployment always has two parts:

1. Push the desired code to the application repository.
2. Build and publish a new image, then point the correct Azure Container App at that image.

## Preconditions

Before deploying, ensure that:

1. Your code is committed in the application repository.
2. The target branch is `pre-final` unless the release process changes.
3. You know which service changed.
4. You are ready to use a brand-new image tag.

## Standard Cloud Shell Refresh

Run this before any deployment:

```bash
cd ~/TestManagement-Tool
git fetch origin
git checkout pre-final
git pull origin pre-final
git rev-parse HEAD
```

## Deployment Decision Matrix

| Change Location | Service To Rebuild | Dockerfile | Azure Container App |
|---|---|---|---|
| Root files such as `server.py`, `dashboard.html`, root `Dockerfile` | `mlb-testplan` | `Dockerfile` | `mlb-testplan` |
| Files under `jira-mcp/` | `jira-mcp` | `jira-mcp/Dockerfile` | `jira-mcp` |
| Files under `confluence-mcp/` | `confluence-mcp` | `confluence-mcp/Dockerfile` | `confluence-mcp` |

## Deploy Root Application Changes

Use this when the dashboard or root API changed.

```bash
cd ~/TestManagement-Tool
git fetch origin
git checkout pre-final
git pull origin pre-final
az acr build -r mjmlbtestplanacr2026 -t mlb-testplan:vNEXT -f Dockerfile .
az containerapp update \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --image mjmlbtestplanacr2026.azurecr.io/mlb-testplan:vNEXT
```

## Deploy Jira MCP Changes

Use this when files under `jira-mcp/` changed.

```bash
cd ~/TestManagement-Tool
git fetch origin
git checkout pre-final
git pull origin pre-final
az acr build -r mjmlbtestplanacr2026 -t jira-mcp:vNEXT -f jira-mcp/Dockerfile .
az containerapp update \
  --name jira-mcp \
  --resource-group mlb-testplan-mj-rg \
  --image mjmlbtestplanacr2026.azurecr.io/jira-mcp:vNEXT
```

## Deploy Confluence MCP Changes

Use this when files under `confluence-mcp/` changed.

```bash
cd ~/TestManagement-Tool
git fetch origin
git checkout pre-final
git pull origin pre-final
az acr build -r mjmlbtestplanacr2026 -t confluence-mcp:vNEXT -f confluence-mcp/Dockerfile .
az containerapp update \
  --name confluence-mcp \
  --resource-group mlb-testplan-mj-rg \
  --image mjmlbtestplanacr2026.azurecr.io/confluence-mcp:vNEXT
```

## Version Tagging Guidance

Use monotonically increasing, never-reused tags.

Examples:

- `mlb-testplan:v4` to `mlb-testplan:v5`
- `jira-mcp:v2` to `jira-mcp:v3`
- `confluence-mcp:v2` to `confluence-mcp:v3`

Avoid reusing old tags because it makes revision tracing and rollback analysis harder.

## Recommended Verification After Every Deployment

### Verify the image actually changed

```bash
az containerapp show \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --query "properties.template.containers[].image" \
  -o tsv
```

Repeat the same pattern for `jira-mcp` or `confluence-mcp` when deploying those services.

### List revisions

```bash
az containerapp revision list \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --query "[].{name:name,active:properties.active,createdTime:properties.createdTime,healthState:properties.healthState}" \
  -o table
```

### Inspect traffic routing

```bash
az containerapp show \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --query "properties.configuration.ingress.traffic" \
  -o json
```

## Roll-Forward Mindset

If the environment does not reflect the change:

1. Confirm the branch and commit in Cloud Shell.
2. Confirm the new image build completed successfully.
3. Confirm the Container App points to the new image tag.
4. Confirm the new revision is active and healthy.
5. Only after that, suspect browser cache or client-side caching.

## Future Change Checklist

Use this checklist for every future deployment:

1. Identify which service changed.
2. Pull latest code in Cloud Shell.
3. Build a new tag for that service.
4. Update the matching Container App.
5. Inspect deployed image and revisions.
6. Validate the public URL.
7. Record the deployed tag somewhere durable if the team needs release history.
