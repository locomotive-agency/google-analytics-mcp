# Google Analytics MCP Server (Experimental)

[![PyPI version](https://img.shields.io/pypi/v/analytics-mcp.svg)](https://pypi.org/project/analytics-mcp/)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![GitHub branch check runs](https://img.shields.io/github/check-runs/googleanalytics/google-analytics-mcp/main)](https://github.com/googleanalytics/google-analytics-mcp/actions?query=branch%3Amain++)
[![PyPI - Downloads](https://img.shields.io/pypi/dm/analytics-mcp)](https://pypi.org/project/analytics-mcp/)
[![GitHub stars](https://img.shields.io/github/stars/googleanalytics/google-analytics-mcp?style=social)](https://github.com/googleanalytics/google-analytics-mcp/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/googleanalytics/google-analytics-mcp?style=social)](https://github.com/googleanalytics/google-analytics-mcp/network/members)
[![YouTube Video Views](https://img.shields.io/youtube/views/PT4wGPxWiRQ)](https://www.youtube.com/watch?v=PT4wGPxWiRQ)

This repo contains the source code for running a local
[MCP](https://modelcontextprotocol.io) server that interacts with APIs for
[Google Analytics](https://support.google.com/analytics).

Join the discussion and ask questions in the
[🤖-analytics-mcp channel](https://discord.com/channels/971845904002871346/1398002598665257060)
on Discord.

## Tools 🛠️

The server uses the
[Google Analytics Admin API](https://developers.google.com/analytics/devguides/config/admin/v1)
and
[Google Analytics Data API](https://developers.google.com/analytics/devguides/reporting/data/v1)
to provide several
[Tools](https://modelcontextprotocol.io/docs/concepts/tools) for use with LLMs.

### Retrieve account and property information 🟠

- `get_account_summaries`: Retrieves information about the user's Google
  Analytics accounts and properties.
- `get_property_details`: Returns details about a property.
- `list_google_ads_links`: Returns a list of links to Google Ads accounts for
  a property.

### Run core reports 📙

- `run_report`: Runs a Google Analytics report using the Data API.
- `get_custom_dimensions_and_metrics`: Retrieves the custom dimensions and
  metrics for a specific property.

### Run realtime reports ⏳

- `run_realtime_report`: Runs a Google Analytics realtime report using the
  Data API.

## Setup instructions 🔧

✨ Watch the [Google Analytics MCP Setup
Tutorial](https://youtu.be/nS8HLdwmVlY) on YouTube for a step-by-step
walkthrough of these instructions.

[![Watch the video](https://img.youtube.com/vi/nS8HLdwmVlY/mqdefault.jpg)](https://www.youtube.com/watch?v=nS8HLdwmVlY)

Setup involves the following steps:

1.  Configure Python.
1.  Configure credentials for Google Analytics.
1.  Configure Gemini.

### Configure Python 🐍

[Install pipx](https://pipx.pypa.io/stable/#install-pipx).

### Enable APIs in your project ✅

[Follow the instructions](https://support.google.com/googleapi/answer/6158841)
to enable the following APIs in your Google Cloud project:

* [Google Analytics Admin API](https://console.cloud.google.com/apis/library/analyticsadmin.googleapis.com)
* [Google Analytics Data API](https://console.cloud.google.com/apis/library/analyticsdata.googleapis.com)

### Configure credentials 🔑

Configure your [Application Default Credentials
(ADC)](https://cloud.google.com/docs/authentication/provide-credentials-adc).
Make sure the credentials are for a user with access to your Google Analytics
accounts or properties.

- **Service account JSON via `GOOGLE_APPLICATION_CREDENTIALS`**: ADC supports
  pointing to a service account key file. Set `GOOGLE_APPLICATION_CREDENTIALS`
  to the absolute path of your service account JSON and the server will use it
  automatically.

- **Delegated subject (domain-wide delegation)**: If your service account needs
  to act on behalf of a user so that
  Analytics properties are visible, set the environment variable
  `ANALYTICS_MCP_SUBJECT` to that user’s email. The server will apply
  domain‑wide delegation when using service account credentials. Ensure the
  service account is configured for
  [delegating domain‑wide authority](https://developers.google.com/identity/protocols/oauth2/service-account#delegatingauthority)
  and has access to the relevant Analytics data. For backward compatibility,
  the alias `GOOGLE_IMPERSONATED_SUBJECT` is also supported.

Credentials must include the Google Analytics read-only scope:

```
https://www.googleapis.com/auth/analytics.readonly
```

Check out
[Manage OAuth Clients](https://support.google.com/cloud/answer/15549257)
for how to create an OAuth client.

Here are some sample `gcloud` commands you might find useful:

- Set up ADC using user credentials and an OAuth desktop or web client after
  downloading the client JSON to `YOUR_CLIENT_JSON_FILE`.

  ```shell
  gcloud auth application-default login \
    --scopes https://www.googleapis.com/auth/analytics.readonly,https://www.googleapis.com/auth/cloud-platform \
    --client-id-file=YOUR_CLIENT_JSON_FILE
  ```

- Set up ADC using service account impersonation.

  ```shell
  gcloud auth application-default login \
    --impersonate-service-account=SERVICE_ACCOUNT_EMAIL \
    --scopes=https://www.googleapis.com/auth/analytics.readonly,https://www.googleapis.com/auth/cloud-platform
  ```

When the `gcloud auth application-default` command completes, copy the
`PATH_TO_CREDENTIALS_JSON` file location printed to the console in the
following message. You'll need this for the next step!

```
Credentials saved to file: [PATH_TO_CREDENTIALS_JSON]
```

### Configure Gemini

1.  Install [Gemini
    CLI](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/index.md)
    or [Gemini Code
    Assist](https://marketplace.visualstudio.com/items?itemName=Google.geminicodeassist).

1.  Create or edit the file at `~/.gemini/settings.json`, adding your server
    to the `mcpServers` list.

    ```json
    {
      "mcpServers": {
        "analytics-mcp": {
          "command": "pipx",
          "args": [
            "run",
            "--spec",
            "git+https://github.com/locomotive-agency/google-analytics-mcp.git",
            "google-analytics-mcp"
          ]
        }
      }
    }
    ```

1.  **Optional:** Configure environment variables in Gemini settings. You may
    want to do this if you always want to use a specific set of credentials or
    delegated subject, regardless of your local ADC.

    In `~/.gemini/settings.json`, add these to the `env` object as needed.

    ```json
    {
      "mcpServers": {
        "analytics-mcp": {
          "command": "pipx",
          "args": [
            "run",
            "--spec",
            "git+https://github.com/locomotive-agency/google-analytics-mcp.git",
            "google-analytics-mcp"
          ],
          "env": {
            "GOOGLE_APPLICATION_CREDENTIALS": "PATH_TO_CREDENTIALS_JSON",
            "GOOGLE_PROJECT_ID": "YOUR_PROJECT_ID"
            "ANALYTICS_MCP_SUBJECT": "client@locomotive.agency"
          }
        }
      }
    }
    ```

## Running on python:slim containers (no git)

If your runtime is a slim Python container without `git`, you can install and
run the server using either the GitHub source ZIP (no git required) or by
including the source in your image.

- **Install from GitHub ZIP (no git):**

  ```bash
  pip install --no-cache-dir https://github.com/locomotive-agency/google-analytics-mcp/archive/refs/heads/main.zip
  ```

- **Install from local source (COPY into image):**

  ```bash
  pip install --no-cache-dir .
  ```

- **Run command:**

  ```bash
  google-analytics-mcp
  ```

- **Required environment variables:**
  - `GOOGLE_APPLICATION_CREDENTIALS`: Absolute path to your service account
    JSON (mount or secret). Example: `/var/secrets/adc.json`.
  - `ANALYTICS_MCP_SUBJECT` (optional): Delegated user email to impersonate,
    e.g. `client@locomotive.agency`.

Example Dockerfile snippet:

```dockerfile
FROM python:3.12-slim

# Install the server without git using the GitHub source ZIP
RUN pip install --no-cache-dir \
    https://github.com/locomotive-agency/google-analytics-mcp/archive/refs/heads/main.zip

# Provide credentials at runtime (recommended: mount a secret)
ENV GOOGLE_APPLICATION_CREDENTIALS=/var/secrets/adc.json
# Optional: impersonate a delegated subject
# ENV ANALYTICS_MCP_SUBJECT=client@locomotive.agency

CMD ["google-analytics-mcp"]
```

## Try it out :lab_coat:

Launch Gemini Code Assist or Gemini CLI and type `/mcp`. You should see
`analytics-mcp` listed in the results.

Here are some sample prompts to get you started:

- Ask what the server can do:

  ```
  what can the analytics-mcp server do?
  ```

- Ask about a Google Analytics property

  ```
  Give me details about my Google Analytics property with 'xyz' in the name
  ```

- Prompt for analysis:

  ```
  what are the most popular events in my Google Analytics property in the last 180 days?
  ```

- Ask about signed-in users:

  ```
  were most of my users in the last 6 months logged in?
  ```

- Ask about property configuration:

  ```
  what are the custom dimensions and custom metrics in my property?
  ```

## Contributing ✨

Contributions welcome! See the [Contributing Guide](CONTRIBUTING.md).
