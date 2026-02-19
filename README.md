# Cribl Inventory

This pack uses Cribl Search **HTTP API Dataset** providers to pull data from the Cribl Stream/Edge API. It gives you a single place to see worker groups, routes, pipelines, packs, inputs, outputs, and an **Edge Node Statistics** dashboard (top 10 by bytes in and by events in, filterable by Fleet).

## What You Get

- **Worker groups** – List of groups/fleets from the Leader API
- **Stream inventory** – Routes, pipelines, packs, inputs, outputs per worker group, using a variabilized `${worker_group}` URL
- **Edge Node Statistics** – Dashboard: Fleet filter; line charts (in/out events and bytes, one point per report per host); two tables—top 10 by bytes in and top 10 by events in; from **cribl_worker_metrics** (Leader `master/workers` API)

## Deployment Overview

You will create **three dataset providers** and **three datasets** (plus one optional for pack details):

| Provider               | Dataset                | Purpose |
|------------------------|------------------------|--------|
| cribl_worker_groups    | cribl_worker_groups    | Groups/fleets list (Fleet dropdown in Edge Node Statistics; Stream Configuration, Pack Information) |
| cribl_stream_inventory | cribl_stream_inventory | Config per `${worker_group}` (routes, pipelines, packs, inputs, outputs) |
| cribl_metrics          | cribl_worker_metrics   | Leader `master/workers` – **Edge Node Statistics** (metrics-over-time line charts, top 10 by bytes in, top 10 by events in) |

---

## Step 1: OAuth (used by all providers)

Create API credentials with admin permissions: [Cribl Cloud API](https://docs.cribl.io/api#criblcloud). You will use the same OAuth settings for every provider below.

- **Login url**: `https://login.cribl.cloud/oauth/token`
- **client secret parameter**: `client_secret`  
- **client secret value**: your API credential secret
- **Extra auth parameters**: `audience` = `https://api.cribl.cloud`, `client_id` = your client_id, `grant_type` = `client_credentials`
- **token_attribute**: `access_token`
- **Authorization header**: `Authorization`
- **authorize expression**: `Bearer ${token}`

---

## Step 2: Provider and Dataset – Worker Groups

- **Data → Dataset providers** → **Generic HTTP API** named **cribl_worker_groups**.
- **Endpoint**: name `cribl_groups`, datafield `items`, method get, url `https://<workspace>-<org>.cribl.cloud/api/v1/master/groups`
- OAuth: use the settings from Step 1.
- **Data → Datasets** → create **cribl_worker_groups**, provider **cribl_worker_groups**, enable **cribl_groups**, save.

This dataset drives the **Fleet** dropdown on the Edge Node Statistics dashboard (groups with `isFleet==true`) and the Stream Configuration and Pack Information dashboards.

---

## Step 3: Provider and Dataset – Stream Inventory (config)

- **Data → Dataset providers** → **Generic HTTP API** named **cribl_stream_inventory**.
- Add these endpoints (same OAuth; replace `<workspace>-<org>` with your base):

| name          | datafield | method | url |
|---------------|-----------|--------|-----|
| cribl_routes  | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/routes |
| cribl_pipelines| items    | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/pipelines |
| cribl_packs   | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/packs |
| cribl_inputs  | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/system/inputs?includePacks=true |
| cribl_outputs | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/system/outputs?includePacks=true |

- **Data → Datasets** → create **cribl_stream_inventory**, provider **cribl_stream_inventory**, enable all five endpoints, add the pack’s **cribl_stream_inventory** datatype ruleset.

---

## Step 4: Provider and Dataset – Metrics (Edge Node Statistics)

- **Data → Dataset providers** → **Generic HTTP API** named **cribl_metrics**.
- **Endpoint**: name `cribl_worker_metrics`, datafield `items`, method get, url `https://<workspace>-<org>.cribl.cloud/api/v1/master/workers`
- OAuth: use the settings from Step 1.
- **Data → Datasets** → create **cribl_worker_metrics**, provider **cribl_metrics**, enable **cribl_worker_metrics**, add the pack’s **cribl_worker_metrics** datatype ruleset.

The Leader `master/workers` API returns workers from all groups (Stream worker groups and Edge fleets). The dashboard expects fields such as `group`, `id`, `info.hostname`, `lastMsgTime`, `status`. Throughput uses **lastMetrics** with bracket syntax: `lastMetrics["total.in_events"]`, `lastMetrics["total.out_events"]`, `lastMetrics["total.in_bytes"]`, `lastMetrics["total.out_bytes"]`, and falls back to flat camelCase/snake_case if your API returns those. If your API uses a different array key than `items` (e.g. `workers`), set **datafield** accordingly.

---

## Edge Node Statistics Dashboard

The pack adds the **Edge Node Statistics** dashboard. Data comes from **cribl_worker_metrics** (Leader `master/workers` API).

- **Time Range** – Picker sets the time window for dataset refresh.
- **Metrics over time** – Four line charts (In events, Out events, In bytes, Out bytes). Each point is one time a host reported; one line per host. No aggregation (e.g. 10 reports in a day → 10 points for that host).
- **Fleet** – Dropdown filters by Edge fleet (from cribl_worker_groups, `isFleet==true`). Choose * for all fleets.
- **Tables** – Top 10 Edge nodes by bytes in (host, id, in_bytes, out_bytes, lastMsgTime) and top 10 by events in (host, id, in_events, out_events, lastMsgTime). When metrics are not available, sorting uses lastMsgTime.

---

## Notes

- If you rename any dataset, update the corresponding macro in the pack (cribl_worker_groups, cribl_stream_inventory, cribl_worker_metrics).

---

## Optional: Pack Details Dashboard

To show inputs/outputs/routes/pipelines for a **selected pack**:

- Create **Generic HTTP API** provider **cribl_packs** (same OAuth) with endpoints:

| name                | datafield | method | url |
|---------------------|-----------|--------|-----|
| cribl_packs_inputs  | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/inputs |
| cribl_packs_outputs | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/outputs |
| cribl_packs_routes  | items     | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/routes |
| cribl_packs_pipelines| items    | get    | https://&lt;workspace&gt;-&lt;org&gt;.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/pipelines |

- Create dataset **cribl_packs** with provider **cribl_packs**, enable those endpoints, add the pack’s **cribl_packs** ruleset. Update the **cribl_packs** macro if you change the dataset name.

---

## Release Notes

| Version | Date       | Changes |
|---------|------------|--------|
| 1.1.7   | 2026-02-17 | Edge Node Statistics: added metrics-over-time line charts (one point per report, one line per host; in/out events and bytes). |
| 1.1.6   | 2026-02-17 | Edge Node Statistics: two tables only (top 10 by bytes in, top 10 by events in); Time Range and Fleet filter; removed Data Activity / line charts. |
| 1.1.2   | 2026-02-17 | Edge Node Statistics: time picker; both tables use **cribl_worker_metrics**. |
| 1.1.1   | 2026-01-27 | Correct Search syntax for throughput metrics (`lastMetrics["total.*"]`). |
| 1.0.1   | 2026-01-27 | Typos and instruction clarifications. |
| 0.9.1   | 2025-12-19 | Beta release. |

---

## Contributing

Reach out to Kelsey Prior (cribl.io) on [Cribl Community Slack](https://cribl-community.slack.com).

## Contact

kprior@cribl.io

## License

[Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE)
