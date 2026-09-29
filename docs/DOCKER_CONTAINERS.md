# Docker Container Inventory

This document captures the current Docker container layout used by the MLB TestPlan toolchain in local and build-oriented workflows.

## Compose Services

| Compose Service | Container Name | Exposed Port | Purpose |
|---|---|---:|---|
| `mlb-testplan` | `mlb-testplan-mcp` | `8080` | Main dashboard and root API |
| `jira-mcp` | `mlb-jira-mcp` | internal `8000` | Jira and Xray proxy |
| `confluence-mcp` | `mlb-confluence-mcp` | internal `8001` | Confluence proxy |

## Compose Network

All services join the same bridge network:

```text
mlb-network
```

This allows the root service to address the helper services by Compose service name.

## Internal Service Wiring

The root service talks to the helper services using these runtime values:

```text
XRAY_BASE_URL=http://jira-mcp:8000
LOCAL_API_URL=http://confluence-mcp:8001
```

This is the local container equivalent of the multi-service Azure deployment.

## Current Container Environment Variables

### `jira-mcp`

```text
PORT=8000
JIRA_PAT=${JIRA_PAT}
JIRA_BASE_URL=${JIRA_BASE_URL:-https://api.skyway.porsche.com/jira}
HTTP_PROXY=${HTTP_PROXY:-}
HTTPS_PROXY=${HTTPS_PROXY:-}
CERT_PATH=${JIRA_CERT_PATH:-}
CERT_PASSWORD=${JIRA_CERT_PASSWORD:-}
```

### `confluence-mcp`

```text
PORT=8001
CONFLUENCE_PAT=${CONFLUENCE_PAT}
CONFLUENCE_BASE_URL=${CONFLUENCE_BASE_URL:-https://api.skyway.porsche.com/confluence}
HTTP_PROXY=${HTTP_PROXY:-}
HTTPS_PROXY=${HTTPS_PROXY:-}
CERT_PATH=${CONFLUENCE_CERT_PATH:-}
CERT_PASSWORD=${CONFLUENCE_CERT_PASSWORD:-}
```

### `mlb-testplan`

```text
XRAY_BASE_URL=http://jira-mcp:8000
LOCAL_API_URL=http://confluence-mcp:8001
JIRA_API_TOKEN=${JIRA_PAT}
CONFLUENCE_PAGE_ID=${CONFLUENCE_PAGE_ID:-2378907792}
```

## Practical Meaning

1. `mlb-testplan` is not a standalone service in practice.
2. If either helper container is missing or unhealthy, parts of the dashboard will fail.
3. Local Docker topology mirrors the logical service split used in Azure Container Apps.

## Useful Local Docker Commands

### Start the stack

```bash
docker-compose up -d
```

### Show running containers

```bash
docker ps
```

### Follow root app logs

```bash
docker logs -f mlb-testplan-mcp
```

### Follow Jira MCP logs

```bash
docker logs -f mlb-jira-mcp
```

### Follow Confluence MCP logs

```bash
docker logs -f mlb-confluence-mcp
```

### Stop the stack

```bash
docker-compose down
```

## Relationship To Azure

The Docker setup is the closest local representation of the Azure deployment model:

1. Three separate services.
2. Clear inter-service dependencies.
3. Containerized boundaries between dashboard logic, Jira/Xray integration, and Confluence integration.

Use this model when reasoning about which Azure Container App must be rebuilt or redeployed for a given change.
