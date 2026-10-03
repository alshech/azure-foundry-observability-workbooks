# Azure Foundry Observability Workbooks

Collection of reusable **Azure Monitor Workbook** templates for observability of **Azure AI Foundry** agents — token consumption, costs, latency, Microsoft Defender for AI Services, and more to come.

Designed to be quickly reused from one client/project to another: each workbook is a generic JSON template, with no hard-coded dependency on a subscription or workspace.

## Contents

| Workbook | Description | Link |
|---|---|---|
| **Token Consumption** | Token usage per model deployment, estimated Defender for AI cost (real-time + 30-day projection), latency, prompt/generated/cached breakdown, hourly heatmap, top costly requests, callers | [`workbooks/token-consumption/workbook.json`](workbooks/token-consumption/workbook.json) |
| **Defender for AI Threat Insights** | Defender for AI security alerts (jailbreak, prompt injection, credential theft, suspicious IP, wallet attacks, etc.) via Azure Resource Graph — no Log Analytics configuration required | [`workbooks/defender-ai-threat-insights/workbook.json`](workbooks/defender-ai-threat-insights/workbook.json) |
| **API Surface & Request Type Usage** | Traffic breakdown by operation type (agent response creation, embeddings, assistant management), payload size, callers by principal — complements the token view with an "API surface" view | [`workbooks/api-surface-usage/workbook.json`](workbooks/api-surface-usage/workbook.json) |

## Quick start

1. Check the [prerequisites](docs/prerequisites.md) (diagnostic settings, permissions).
2. Azure portal → **Monitor** (or directly on the Foundry resource) → **Workbooks** → **New**.
3. Click the **Advanced Editor** icon (`</>`) in the toolbar.
4. Paste the contents of the desired workbook's JSON file.
5. **Apply** then **Done Editing**.
6. Select your **Log Analytics workspace** in the `Workspace` parameter at the top of the workbook.
7. **Save**, choosing the target resource group.

## CLI deployment (optional)

```bash
az resource create \
  --resource-group <rg> \
  --resource-type "Microsoft.Insights/workbooks" \
  --name <workbook-guid> \
  --location <region> \
  --api-version 2022-04-01 \
  --is-full-object \
  --properties '{
    "location": "<region>",
    "kind": "shared",
    "properties": {
      "displayName": "Foundry Token Consumption",
      "serializedData": "<escaped-json-content-as-string>",
      "category": "workbook",
      "sourceId": "<log-analytics-workspace-resource-id>",
      "version": "1.0"
    }
  }'
```

## Validated data schema

The KQL queries in this repo are built and tested against the **actual schema** observed on an Azure AI Foundry account (`AzureOpenAIRequestUsage` and `RequestResponse` categories of the `AzureDiagnostics` table, legacy mode), not against generic documentation. See [prerequisites](docs/prerequisites.md#5-limites-connues-schéma-vérifié-en-conditions-réelles) for details on known limitations and common pitfalls (JSON arrays, fields missing depending on the account, etc.).

## Positioning relative to native Foundry / Defender for Cloud

These workbooks are designed to **avoid duplicating**:
- Foundry's native **Application analytics** dashboard (`Monitoring` in the Foundry portal) — which requires an Application Insights instance connected to the project and relies on tracing, not on the Cognitive Services account's diagnostic logs.
- Defender for Cloud's generic compliance/posture workbooks — which do not specifically cover AI threat protection alerts.

Each workbook in this repo works **without App Insights tracing**, relying instead on the account's native diagnostic logs (`AzureDiagnostics`) or on Azure Resource Graph.

## Contributing

Contributions are welcome: new workbooks, query improvements, schema fixes. Please add any new workbook in its own folder under `workbooks/<name>/` with a `workbook.json`, and document its specific prerequisites if they differ from the repo's.

## License

[MIT](LICENSE)
