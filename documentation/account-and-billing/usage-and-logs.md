---
description: Track your requests and spending in the dashboard and export request logs.
---

# Usage and logs

This page explains how to monitor your requests and spending in the Engy dashboard. It is intended for users who track costs or troubleshoot requests.

## Overview

The dashboard provides two views of your activity:

| Page | Purpose | Time range |
| --- | --- | --- |
| **Usage** | Aggregated requests and spending per day and per model | Last 7 or 30 days |
| **Logs** | Individual requests with token counts, latency, and cost | Last 1 or 3 days |

To retrieve the same usage data programmatically, use the [Management API](management-api.md).

## Usage

The **Usage** page shows a chart and a table of requests and spending, broken down by model.

Usage data is aggregated per UTC day and covers complete days only. Requests from the current day appear after the day ends.

## Logs

The **Logs** page lists individual requests. You can filter the list by model.

Each entry contains the following fields:

| Field | Description |
| --- | --- |
| Time | The time of the request, in UTC |
| Model | The model ID |
| Input | The number of input tokens |
| Output | The number of output tokens |
| Cached | The number of input tokens billed at the cached rate |
| TTFT | The time to first token |
| Status | The HTTP status code of the request |
| Cost | The amount charged for the request |

### Export logs

To export the logs that match the current filter, use the CSV export on the **Logs** page.
