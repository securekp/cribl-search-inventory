# Cribl Inventory

This pack uses Cribl Search **HTTP API Dataset** providers to pull data from the Cribl Stream/Edge API. It gives you a single place to see worker groups, routes, pipelines, packs, inputs, outputs, and a **Heavy Talkers for Edge** dashboard (top Edge nodes by throughput).

## What You Get

- **Worker groups** – List of groups/fleets from the Leader API
- **Stream inventory** – Routes, pipelines, packs, inputs, outputs per worker group, using a variabilized `${worker_group}` URL
- **Heavy Talkers for Edge** – One dashboard: Edge nodes by fleet with throughput metrics (in/out events and bytes), time picker, and Fleet dropdown

## Deployment Overview

You will create **three dataset providers** and **three datasets** (plus one optional for pack details):

| Provider              | Dataset               | Purpose |
|-----------------------|-----------------------|--------|
| cribl_worker_groups   | cribl_worker_groups   | Groups/fleets list (drives Fleet dropdown in Heavy Talkers) |
| cribl_stream_inventory| cribl_stream_inventory| Config per `${worker_group}` (routes, pipelines, packs, inputs, outputs) |
| cribl_metrics        | cribl_worker_metrics  | Leader `master/workers` – used for **Heavy Talkers for Edge** (Edge nodes, with lastMetrics when available) |

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

This dataset drives the **Fleet** dropdown on the Heavy Talkers for Edge dashboard. Ensure the API returns **isFleet** (or adjust dashboard queries if your field name differs).

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

## Step 4: Provider and Dataset – Metrics (Heavy Talkers for Edge)

- **Data → Dataset providers** → **Generic HTTP API** named **cribl_metrics**.
- **Endpoint**: name `cribl_worker_metrics`, datafield `items`, method get, url `https://<workspace>-<org>.cribl.cloud/api/v1/master/workers`
- OAuth: same as Step 1.
- **Data → Datasets** → create **cribl_worker_metrics**, provider **cribl_metrics**, enable **cribl_worker_metrics**, add the pack’s **cribl_worker_metrics** datatype ruleset.

This dataset feeds **Heavy Talkers for Edge**. The Leader `master/workers` response includes Edge nodes and often **lastMetrics** (throughput: in/out events and bytes) for them.

---

## Heavy Talkers for Edge Dashboard

The pack adds the **Heavy Talkers for Edge** dashboard.

- **Time Range** – Picker controls the time window (`$time_range.earliest$` / `$time_range.latest$`).
- **Fleet** – Dropdown filters by Edge fleet (groups with `isFleet==true`). Choose * for all fleets. Data from **cribl_worker_metrics** (Leader `master/workers`).

The table shows top Edge nodes by outbound events (or by last activity when metrics are missing). Throughput columns use **lastMetrics** with bracket syntax: `lastMetrics["total.in_events"]`, `lastMetrics["total.out_events"]`, `lastMetrics["total.in_bytes"]`, `lastMetrics["total.out_bytes"]`. The dashboard also accepts flat camelCase/snake_case if your API returns those.

---

## Notes

- If your API uses a different array key than `items` (e.g. `workers`), set **datafield** accordingly for that endpoint.
- Heavy Talkers for Edge expects fields such as `group`, `id`, `info.hostname`, `lastMsgTime`, `status`; throughput comes from **lastMetrics** when present.
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
| 1.1.3   | 2026-02-17 | **Heavy Talkers for Edge** only: renamed dashboard, removed Stream/Worker table and Worker Group dropdown; README and versioning updated. |
| 1.1.2   | 2026-02-17 | Heavy Talkers: time picker; in/out column order; both tables use **cribl_worker_metrics** (Worker table filters by group; avoids 404 when `/m/{group}/workers` not available). |
| 1.1.1   | 2026-01-27 | Heavy Talkers: correct Search syntax for throughput metrics (`lastMetrics["total.*"]`). |
| 1.0.1   | 2026-01-27 | Typos and instruction clarifications. |
| 0.9.1   | 2025-12-19 | Beta release. |

---

## Contributing

Reach out to Kelsey Prior (cribl.io) on [Cribl Community Slack](https://cribl-community.slack.com).

## Contact

kprior@cribl.io

## License

[Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE)
