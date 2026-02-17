# Cribl Inventory

This pack helps you see your Cribl infrastructure, configurations, and **node metrics** in one place. It displays worker groups, routes, pipelines, packs, and a **Heavy Talkers** dashboard for Edge and Worker nodes.

## About this Pack

- Uses the **HTTP API Dataset** provider in Cribl Search to pull data from the Cribl Stream/Edge API
- **Configuration**: worker groups, routes, pipelines, packs, inputs, outputs
- **Metrics**: worker/edge node list for **Heavy Talkers** (top nodes by throughput), filterable by **Fleet** or **Worker group**
- Helps new admins see what’s configured and which nodes are busiest

## Deployment

### 1. Configure Dataset Provider for Worker Groups

- Go to **Data → Dataset providers**.
- Create a new **Generic HTTP API** provider named **cribl_worker_groups**.
- Add one endpoint:
  - **name**: `cribl_groups`
  - **datafield**: `items`
  - **method**: get
  - **url**: `https://<workspace>-<org>.cribl.cloud/api/v1/master/groups`
- **Authorization** tab: **OAuth**
  - Create API credentials with admin permissions: [Cribl Cloud API](https://docs.cribl.io/api#criblcloud)
  - **Login url**: `https://login.cribl.cloud/oauth/token`
  - **client secret parameter**: `client_secret`
  - **client secret value**: your API credential secret
  - **Extra auth parameters** (exact names):
    - `audience` = `https://api.cribl.cloud`
    - `client_id` = your API client_id
    - `grant_type` = `client_credentials`
  - **token_attribute**: `access_token`
  - **Authorization header**: `Authorization`
  - **authorize expression**: `Bearer ${token}`

### 2. Configure Dataset Provider for Configuration Items

- Create a **Generic HTTP API** provider named **cribl_stream_inventory**.
- Add these endpoints (same OAuth as above; replace `<workspace>-<org>` in base URL):

| name           | datafield | method | url |
|----------------|-----------|--------|-----|
| cribl_routes   | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/routes |
| cribl_pipelines| items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/pipelines |
| cribl_packs    | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/packs |
| cribl_inputs   | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/system/inputs?includePacks=true |
| cribl_outputs  | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/system/outputs?includePacks=true |

- Use the same **OAuth** settings as in step 1.

### 3. Configure Dataset Provider for Metrics (Heavy Talkers)

- Create a **Generic HTTP API** provider named **cribl_metrics**.
- Add one endpoint:
  - **name**: `cribl_worker_metrics`
  - **datafield**: `items`
  - **method**: get
  - **url**: `https://<workspace>-<org>.cribl.cloud/api/v1/master/workers`
- Use the same **OAuth** settings as in step 1.

> **Note:** The Cribl API returns workers from all groups (Stream worker groups and Edge fleets). The response may use `items` or another array key; if your API uses a different key (e.g. `workers`), set **datafield** to that key. The **Heavy Talkers** dashboard expects fields such as `group`, `id`, `info.hostname`, `lastMsgTime`, `status`. Throughput metrics in Search typically come from **`lastMetrics.total`**: `lastMetrics.total.in_events`, `lastMetrics.total.out_events`, `lastMetrics.total.in_bytes`, `lastMetrics.total.out_bytes`. The dashboard uses these first and falls back to flat camelCase/snake_case if present.

### 4. Configure Cribl Worker Groups Dataset

- **Data → Datasets**: create **cribl_worker_groups**.
  - **Dataset provider**: cribl_worker_groups
  - **Enabled endpoints**: cribl_groups
- If you use a different dataset name, update the **cribl_worker_groups** macro in the pack to match.

### 5. Configure Cribl Inventory Dataset

- Create dataset **cribl_stream_inventory**.
  - **Dataset provider**: cribl_stream_inventory
  - **Enabled endpoints**: cribl_routes, cribl_pipelines, cribl_packs, cribl_inputs, cribl_outputs
  - **Processing**: add the pack’s **cribl_stream_inventory** datatype ruleset
- If you use a different name, update the **cribl_stream_inventory** macro.

### 6. Configure Cribl Worker Metrics Dataset

- Create dataset **cribl_worker_metrics**.
  - **Dataset provider**: cribl_metrics
  - **Enabled endpoints**: cribl_worker_metrics
  - **Processing**: add the pack’s **cribl_worker_metrics** datatype ruleset
- If you use a different name, update the **cribl_worker_metrics** macro.

### 7. Heavy Talkers Dashboard

- The pack adds the **Heavy Talkers – Edge & Worker Nodes** dashboard.
- **Fleet (Edge)** dropdown: filter Edge heavy talkers by fleet (uses groups where `isFleet==true`).
- **Worker Group (Stream)** dropdown: filter Worker heavy talkers by worker group (`isFleet==false`).
- Tables show top nodes by outbound events (or by `lastMsgTime` if event metrics are not in the API).
- Ensure the **cribl_worker_groups** dataset exposes **isFleet** so Fleet vs Worker group filters work. If your `/api/v1/master/groups` response uses a different field name, adjust the dashboard queries accordingly.

---

### OPTIONAL: Dataset Provider and Dataset for Pack Details

To show inputs/outputs/routes/pipelines for a **selected pack** (second dashboard):

- Create **Generic HTTP API** provider **cribl_packs** with OAuth as above and endpoints:

| name               | datafield | method | url |
|--------------------|-----------|--------|-----|
| cribl_packs_inputs  | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/inputs |
| cribl_packs_outputs | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/outputs |
| cribl_packs_routes  | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/routes |
| cribl_packs_pipelines| items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/pipelines |

- Create dataset **cribl_packs** with provider **cribl_packs**, enable those endpoints, and add the pack’s **cribl_packs** processing ruleset. Update the **cribl_packs** macro if you change the dataset name.

## Release Notes
### Version 1.0.1 - 2026-01-27
Fix a couple of typos and clarify some instructions.

### Version 0.9.1 - 2025-12-19
Beta Release

- Beta release.

## Contributing

To contribute or report issues, reach out to Kelsey Prior (cribl.io) on [Cribl Community Slack](https://cribl-community.slack.com).

## Contact

kprior@cribl.io

## License
This Pack uses the following license: [`Apache 2.0`](https://github.com/criblio/appscope/blob/master/LICENSE).
