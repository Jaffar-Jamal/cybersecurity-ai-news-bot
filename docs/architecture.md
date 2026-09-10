# Architecture

## Overview

The Cybersecurity AI News Bot is a small integration project that connects a public cybersecurity RSS feed to an AI summarization service and a messaging platform.

```text
+----------------------+
| BleepingComputer RSS |
+----------+-----------+
           |
           v
+----------------------+
|         n8n          |
|  Workflow Orchestrator|
+----------+-----------+
           |
           v
+----------------------+
|      Limit Node      |
+----------+-----------+
           |
           v
+----------------------+
|   Google Gemini API  |
| Summary + Severity   |
+----------+-----------+
           |
           v
+----------------------+
|     Edit Fields      |
|  Clean Output Only   |
+----------+-----------+
           |
           v
+----------------------+
| WhatsApp Cloud API   |
+----------+-----------+
           |
           v
+----------------------+
| WhatsApp Recipient   |
+----------------------+
```

## Data Flow

### 1. RSS ingestion

The RSS node retrieves structured article data from:

`https://www.bleepingcomputer.com/feed/`

Relevant fields include the article title, content, publication date, and source URL.

### 2. Item limiting

The Limit node restricts the number of articles processed during testing.

This was especially useful while working within external API rate limits.

### 3. AI processing

The Google Gemini node receives the article title, body content, and source URL.

The prompt asks Gemini to produce:

- A concise technical summary
- Why the story matters
- A severity rating
- The original source URL

### 4. Output transformation

Gemini's response includes additional response structure and metadata.

The Edit Fields node extracts only:

`content.parts[0].text`

and stores it as:

`Summary`

This gives the downstream messaging node a clean, predictable field.

### 5. WhatsApp delivery

The WhatsApp Business Cloud node sends the final `Summary` field as a text message.

The public workflow contains placeholder account values only. Credentials must be created separately inside n8n.

## Hosting

The n8n instance was self-hosted on an Ubuntu VPS using Docker.

Traefik was used as the reverse proxy and for HTTPS/TLS termination.

The environment required troubleshooting around DNS, hostname configuration, certificate issuance, and container recreation after configuration changes.

## Security Design

Secrets are not stored in the public workflow export.

The public repository intentionally excludes:

- API keys
- Bearer tokens
- Credential objects
- Personal phone numbers
- WhatsApp account identifiers
- n8n instance metadata
- Private environment files

## Future Architecture

A stronger production-style version would change the workflow from processing each story independently to:

```text
RSS
 |
 v
Select 5 stories
 |
 v
Aggregate stories
 |
 v
One Gemini request
 |
 v
One daily briefing
 |
 v
WhatsApp
```

This would reduce API usage and create a cleaner user experience.
