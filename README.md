# Cribl Inventory

This pack uses Cribl Search **HTTP API Dataset** providers to pull data from the Cribl Stream/Edge API. It gives you a single place to see worker groups, routes, pipelines, packs, inputs, outputs, and an **Edge Node Statistics** dashboard (fleet-wide KPIs, bar charts, composition pie charts, and top 10 tables, filterable by Fleet).

## What You Get

- **Worker groups** – List of groups/fleets from the Leader API
- **Stream inventory** – Routes, pipelines, packs, inputs, outputs per worker group, using a variabilized `${worker_group}` URL
- **Edge Node Statistics** – Fleet-scoped worker metrics from **cribl_worker_metrics**; see **Edge Node Statistics** below for the dashboard and setup.

## Deployment Overview

You will create **three dataset providers** and **three datasets** (plus one optional for pack-scoped config). Each piece is documented in its own section below.

**What you’ll create**

- **cribl_worker_groups** (provider + dataset) — Groups/fleets from the Leader API.
- **cribl_stream_inventory** (provider + dataset) — Per–worker-group config (routes, pipelines, packs, inputs, outputs).
- **cribl_metrics** (provider) + **cribl_worker_metrics** (dataset) — Leader `master/workers` for **Edge Node Statistics**.
- **Optional:** **cribl_packs** (provider + dataset) — Pack-scoped inputs, outputs, routes, pipelines.

**Suggested order**

1. **OAuth** (shared by all HTTP API providers)  
2. **Worker groups** → **Stream inventory** → **Edge Node Statistics** (metrics)  
3. **Pack Details** (optional)

---

## OAuth (all providers)

Every **Generic HTTP API** provider in this pack uses the same OAuth client-credentials flow against Cribl Cloud. Create API credentials with admin permissions: [Cribl Cloud API](https://docs.cribl.io/api#criblcloud).

**Purpose**

- Obtain a bearer token so Search can call Leader and managed-worker (`/m/${worker_group}/...`) endpoints on your behalf.

**How to configure**

Use these settings on each provider’s OAuth configuration:

- **Login url**: `https://login.cribl.cloud/oauth/token`
- **client secret parameter**: `client_secret`
- **client secret value**: your API credential secret
- **Extra auth parameters**: `audience` = `https://api.cribl.cloud`, `client_id` = your client_id, `grant_type` = `client_credentials`
- **token_attribute**: `access_token`
- **Authorization header**: `Authorization`
- **authorize expression**: `Bearer ${token}`

---

## Worker groups

The **cribl_worker_groups** dataset is a single call to the Leader **groups** API. It lists worker groups and fleets (including which are Edge fleets).

**What it powers**

- **Edge Node Statistics** — Fleet dropdown (groups with `isFleet==true`).
- **Stream Configuration** and **Pack Information** — Worker group / fleet picker.

**How to set it up**

- **Data → Dataset providers** → **Generic HTTP API** named **cribl_worker_groups**.
- **Endpoint**: name `cribl_groups`, datafield `items`, method get, url `https://<workspace>-<org>.cribl.cloud/api/v1/master/groups`
- OAuth: use **OAuth (all providers)** above.
- **Data → Datasets** → create **cribl_worker_groups**, provider **cribl_worker_groups**, enable **cribl_groups**, save.

---

## Stream inventory (config)

The **cribl_stream_inventory** dataset exposes Stream/Edge **configuration** per selected worker group. URLs include `${worker_group}` so dashboards can switch group without new provider rows.

**What it powers**

- Dashboards that list routes, pipelines, packs, system inputs, and system outputs for the chosen group.

**How to set it up**

1. **Data → Dataset providers** → **Generic HTTP API** named **cribl_stream_inventory**.
2. OAuth: use **OAuth (all providers)** above.
3. Add these endpoints (replace `<workspace>-<org>` with your base):

- **cribl_routes** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/routes`
- **cribl_pipelines** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/pipelines`
- **cribl_packs** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/packs`
- **cribl_inputs** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/system/inputs?includePacks=true`
- **cribl_outputs** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/system/outputs?includePacks=true`

4. **Data → Datasets** → create **cribl_stream_inventory**, provider **cribl_stream_inventory**, enable all five endpoints, add the pack’s **cribl_stream_inventory** datatype ruleset.

---

## Edge Node Statistics

The pack adds an **Edge Node Statistics** dashboard. It shows a snapshot of Edge fleet workers from the Leader **`master/workers`** API: fleet-wide throughput totals, top hosts by bytes and events (bars and tables), and how traffic splits across hosts (pie charts). Data is **just-in-time** from the API when you run the dashboard—not a historical time series.

**What’s on the dashboard**

- **Time Range** – Window used when the dataset is queried (refresh).
- **Fleet** – Filter by Edge fleet (from **cribl_worker_groups**, `isFleet==true`). Use * for all fleets. Requires **Worker groups** above.
- **Fleet-wide KPIs** – Four counters: total In events, In bytes, Out events, Out bytes for the selected fleet.
- **Bar charts** – Top 10 Edge nodes by bytes in and by events in.
- **Composition** – Share of in bytes and in events by host (pie charts).
- **Tables** – Top 10 by bytes in (host, id, in_bytes, out_bytes, lastMsgTime) and top 10 by events in (host, id, in_events, out_events, lastMsgTime). If metrics are missing, sorting falls back to lastMsgTime.

**How to set it up**

1. Complete **Worker groups** so the Fleet dropdown works (**cribl_worker_groups**).
2. Add the metrics dataset the dashboard queries:
   - **Data → Dataset providers** → **Generic HTTP API** named **cribl_metrics**.
   - **Endpoint**: name `cribl_worker_metrics`, datafield `items`, method get, url `https://<workspace>-<org>.cribl.cloud/api/v1/master/workers`
   - OAuth: same as **OAuth (all providers)**.
   - **Data → Datasets** → create **cribl_worker_metrics**, provider **cribl_metrics**, enable **cribl_worker_metrics**, add the pack’s **cribl_worker_metrics** datatype ruleset.

---

## Pack Details (optional)

Optional provider and dataset for dashboards that drill into **one pack** under a worker group: that pack’s inputs, outputs, routes, and pipelines.

**What it powers**

- Pack-scoped config views (URLs include **`${worker_group}`** and **`${pack}`**).

**How to set it up**

1. **Data → Dataset providers** → **Generic HTTP API** named **cribl_packs**.
2. OAuth: use **OAuth (all providers)** above.
3. Add endpoints (replace `<workspace>-<org>`):

- **cribl_packs_inputs** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/inputs`
- **cribl_packs_outputs** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/outputs`
- **cribl_packs_routes** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/routes`
- **cribl_packs_pipelines** — datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/pipelines`

4. **Data → Datasets** → create **cribl_packs**, provider **cribl_packs**, enable those endpoints, add the pack’s **cribl_packs** ruleset. Update the **cribl_packs** macro if you rename the dataset.

---

## Macros and renames

If you rename any dataset, update the corresponding macro in the pack (**cribl_worker_groups**, **cribl_stream_inventory**, **cribl_worker_metrics**, **cribl_packs** as applicable).

---

## Release Notes

- **1.1.9** (2026-02-17) — README: list-style deployment/endpoints; Edge Node Statistics + setup in one section; same section pattern across OAuth, worker groups, stream inventory, Edge Node Statistics, Pack Details; macros use `dataset="..."`. Dashboard doc alignment.
- **1.1.8** (2026-02-17) — Edge Node Statistics: fleet-wide KPIs (in/out events, in/out bytes), top-10 bar charts, composition pie charts (share by host); removed line charts (data is just-in-time API snapshot).
- **1.0.1** (2026-01-27) — Typos and instruction clarifications.
- **0.9.1** (2025-12-19) — Beta release.

---

## Contributing

Reach out to Kelsey Prior (cribl.io) on [Cribl Community Slack](https://cribl-community.slack.com).

## Contact

kprior@cribl.io

## License

[Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE)
