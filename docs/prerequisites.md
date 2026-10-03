# Prerequisites

Common prerequisites for using the workbooks in this repo on an Azure AI Foundry account (`Microsoft.CognitiveServices/accounts`).

## 1. Diagnostic settings

Enable a **diagnostic setting** on the Foundry account (not just the project) routed to a **Log Analytics workspace**:

- **`AzureOpenAIRequestUsage`** log category — required for tokens, latency, and cost.
- **`RequestResponse`** category — for callers (IP) and raw request statistics.
- Optional: **`AllMetrics`** — to cross-reference with Metrics Explorer (`GeneratedTokens`, `ProcessedPromptTokens`, `AzureOpenAIRequests`, etc.).

```bash
az monitor diagnostic-settings create \
  --name "all-logs" \
  --resource <account-resource-id> \
  --workspace <log-analytics-workspace-id> \
  --logs '[{"category":"AzureOpenAIRequestUsage","enabled":true},{"category":"RequestResponse","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

## 2. Permissions

| Action | Minimum role |
|---|---|
| Create/modify the diagnostic setting | Contributor or Monitoring Contributor on the Foundry account |
| Read data in the workbook | Reader on the Log Analytics workspace |
| Deploy the workbook | Contributor on the target resource group |

## 3. Real traffic

No data appears until calls have been made to the model deployments. Generate a few test invocations if needed before validating a deployment.

## 4. Adapting the workbook for a client

After import (see README), select your workspace in the **Workspace** parameter at the top of the workbook — no JSON modification is needed for this, the default value is empty.

## 5. Known limitations (schema verified under real conditions)

- `AzureOpenAIRequestUsage` logs identify the **model deployment**, not the calling Foundry agent. If multiple agents share the same deployment, their consumption cannot be distinguished without tracing (Application Insights + OpenTelemetry GenAI on the project side).
- `promptTokens`, `generatedTokens`, `cachedTokens` are **JSON arrays** in `properties_s` — always aggregate with `array_sum()`.
- `statusCode`/`responseCode` are not guaranteed to be populated depending on the account — check `ResultType`/`Level` before relying on them to detect 429/5xx errors.
- `CallerIPAddress` is only available in the `RequestResponse` category, not in `AzureOpenAIRequestUsage`.
