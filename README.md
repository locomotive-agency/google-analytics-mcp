# Google Analytics MCP Server

[![PyPI version](https://img.shields.io/pypi/v/google-analytics-mcp-python.svg)](https://pypi.org/project/google-analytics-mcp-python/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

A Model Context Protocol (MCP) server for comprehensive Google Analytics API access. Built with FastMCP and maintained by [Locomotive Agency](https://locomotive.agency) for use with [mcpanywhere.com](https://mcpanywhere.com).

This server provides access to the [Google Analytics Admin API](https://developers.google.com/analytics/devguides/config/admin/v1) and [Google Analytics Data API](https://developers.google.com/analytics/devguides/reporting/data/v1) through MCP tools for LLM integration.

## Features

The server provides the following MCP tools:

### Account & Property Information

- `get_account_summaries` - Retrieves information about Google Analytics accounts and properties
- `get_property_details` - Returns details about a specific property
- `list_google_ads_links` - Lists Google Ads account links for a property

### Core Reports

- `run_report` - Runs a Google Analytics report using the Data API
- `get_custom_dimensions_and_metrics` - Retrieves custom dimensions and metrics for a property

### Realtime Reports

- `run_realtime_report` - Runs a Google Analytics realtime report using the Data API

## Installation

### Recommended (via uv)

```bash
uv tool install google-analytics-mcp-python
```

### Alternative (via pip)

```bash
pip install google-analytics-mcp-python
```

### Alternative (via pipx)

```bash
pipx install google-analytics-mcp-python
```

## Configuration

### 1. Enable Google Analytics APIs

Enable the following APIs in your Google Cloud project:

- [Google Analytics Admin API](https://console.cloud.google.com/apis/library/analyticsadmin.googleapis.com)
- [Google Analytics Data API](https://console.cloud.google.com/apis/library/analyticsdata.googleapis.com)

### 2. Create Service Account

1. Go to the [Google Cloud Console](https://console.cloud.google.com/)
2. Create a service account with the Analytics API scope
3. Download the JSON key file

### 3. Set Environment Variables

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/service-account-key.json"
```

#### Domain-Wide Delegation (Optional)

If you need to access Analytics properties on behalf of a user:

```bash
export ANALYTICS_MCP_SUBJECT="user@yourdomain.com"
```

**Note:** For backward compatibility, `GOOGLE_IMPERSONATED_SUBJECT` is also supported.

Required OAuth scope:
```
https://www.googleapis.com/auth/analytics.readonly
```

## Usage with MCP Clients

### Claude Desktop / Gemini

Add to your MCP settings file (`~/.gemini/settings.json` or Claude Desktop config):

```json
{
  "mcpServers": {
    "google-analytics": {
      "command": "uvx",
      "args": ["google-analytics-mcp-python"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/service-account-key.json",
        "ANALYTICS_MCP_SUBJECT": "user@yourdomain.com"
      }
    }
  }
}
```

## Docker Deployment

Example Dockerfile for containerized deployment:

```dockerfile
FROM python:3.12-slim

RUN pip install --no-cache-dir google-analytics-mcp-python

ENV GOOGLE_APPLICATION_CREDENTIALS=/var/secrets/service-account.json

CMD ["google-analytics-mcp"]
```

Required environment variables:
- `GOOGLE_APPLICATION_CREDENTIALS` - Path to service account JSON
- `ANALYTICS_MCP_SUBJECT` (optional) - User email for domain-wide delegation

## Example Prompts

Once configured, you can interact with your Google Analytics data:

```
What are the most popular events in my Google Analytics property in the last 180 days?
```

```
Give me details about my Google Analytics property with 'xyz' in the name
```

```
What are the custom dimensions and custom metrics in my property?
```

```
Were most of my users in the last 6 months logged in?
```

## Development

This server is maintained by [Locomotive Agency](https://locomotive.agency) as part of the MCP Anywhere ecosystem.

Original implementation by Google Analytics team. Enhanced with domain-wide delegation support for enterprise use cases.

## License

Apache License 2.0 - See LICENSE file for details.

## Contributing

Contributions welcome! See the [Contributing Guide](CONTRIBUTING.md).
