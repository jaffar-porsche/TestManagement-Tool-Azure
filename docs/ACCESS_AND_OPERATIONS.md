# Access And Operations Guide

This document describes how to access the running services, inspect their state, and troubleshoot common operational issues.

## Primary URLs

| What You Need | URL |
|---|---|
| Main application landing page | `https://mlb-testplan.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/` |
| Main dashboard | `https://mlb-testplan.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/dashboard` |
| Jira MCP API docs | `https://jira-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs` |
| Confluence MCP API docs | `https://confluence-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs` |

## How To Access The Environment

### From a browser

Open the public URLs directly.

### From Azure Cloud Shell

Use the Azure CLI commands in this document and in `docs/DEPLOYMENT_RUNBOOK.md`.

### From a local shell

You can use `curl` to inspect deployed content.

Example:

```bash
curl -s https://mlb-testplan.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/dashboard | grep -n "Update PAT"
```

This is useful when you want to confirm that a UI change is present in the deployed HTML without relying on your browser cache.

## Daily Operational Commands

### Show the current image for a service

```bash
az containerapp show \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --query "properties.template.containers[].image" \
  -o tsv
```

### Show the latest revision state

```bash
az containerapp revision list \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --query "[].{name:name,active:properties.active,createdTime:properties.createdTime,healthState:properties.healthState}" \
  -o table
```

### Show ingress traffic routing

```bash
az containerapp show \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --query "properties.configuration.ingress.traffic" \
  -o json
```

### Stream logs from a Container App

```bash
az containerapp logs show \
  --name mlb-testplan \
  --resource-group mlb-testplan-mj-rg \
  --follow
```

Swap the `--name` value to `jira-mcp` or `confluence-mcp` when needed.

## Common Situations

### Situation: GitHub contains the latest code, but Azure still shows old behavior

Most likely causes:

1. No new image was built.
2. The Container App still points to the old image tag.
3. A different service is stale, especially one of the helper services.
4. The browser is serving cached assets.

Recommended order of checks:

1. Verify the exact image tag deployed.
2. Verify the active revision.
3. Verify the public endpoint with `curl`.
4. Check logs.

### Situation: Dashboard loads but some features fail

The main app depends on `jira-mcp` and `confluence-mcp`. A healthy root dashboard does not guarantee healthy helper services.

Check these endpoints:

- `https://jira-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs`
- `https://confluence-mcp.purpleplant-c6aa08b1.germanywestcentral.azurecontainerapps.io/docs`

Then inspect their deployed image tags and logs.

### Situation: You need to know which service to redeploy

Use this rule:

1. Root UI or root API change: redeploy `mlb-testplan`.
2. Jira, Xray, or issue proxy change: redeploy `jira-mcp`.
3. Confluence page retrieval or parsing proxy change: redeploy `confluence-mcp`.

## Useful Query Variants

### Check Jira MCP image

```bash
az containerapp show \
  --name jira-mcp \
  --resource-group mlb-testplan-mj-rg \
  --query "properties.template.containers[].image" \
  -o tsv
```

### Check Confluence MCP image

```bash
az containerapp show \
  --name confluence-mcp \
  --resource-group mlb-testplan-mj-rg \
  --query "properties.template.containers[].image" \
  -o tsv
```

## Maintenance Guidance

1. Treat this repository as the durable runbook for the environment.
2. Update URLs, registry names, and resource group names as soon as they change.
3. Add newly introduced services to all three documents in this repository.
4. Keep commands copy-pasteable without requiring reconstruction from memory.
