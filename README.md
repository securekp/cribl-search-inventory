# Cribl Inventory

Cribl Search **HTTP API Dataset** providers against the Stream/Edge Leader API: worker groups, per-group config (routes, pipelines, packs, inputs, outputs), and an **Edge Node Statistics** dashboard (fleet KPIs, bar and pie charts, top-10 tables, Fleet filter).

## What You Get

- **Worker groups** (`cribl_worker_groups`) — Groups and fleets from the Leader API; powers Fleet picker and group dropdowns.
- **Stream inventory** (`cribl_stream_inventory`) — Config per `${worker_group}` for routes, pipelines, packs, inputs, outputs.
- **Edge Node Statistics** (`cribl_worker_metrics` via provider **cribl_metrics**) — Worker snapshot from `master/workers`; see **Edge Node Statistics** for the dashboard and setup.
- **Optional — Pack Details** (`cribl_packs`) — Inputs, outputs, routes, pipelines for a selected pack.

## How to deploy (order)

1. **OAuth** — Same credentials on every Generic HTTP API provider below.  
2. **Worker groups** → **Stream inventory** → **Edge Node Statistics** (metrics provider + dataset).  
3. **Pack Details** — Only if you use the pack-scoped dashboards.

Sections below follow that order. Rename a dataset? Update the matching macro (**Macros and renames**).

---

## OAuth (all providers)

Create API credentials with admin permissions: [Cribl Cloud API](https://docs.cribl.io/api#criblcloud). Every provider here uses client credentials to obtain a bearer token for Leader and `/m/${worker_group}/...` calls.

**OAuth settings** (repeat on each provider):

- **Login url**: `https://login.cribl.cloud/oauth/token`
- **client secret parameter**: `client_secret`
- **client secret value**: your API credential secret
- **Extra auth parameters**: `audience` = `https://api.cribl.cloud`, `client_id` = your client_id, `grant_type` = `client_credentials`
- **token_attribute**: `access_token`
- **Authorization header**: `Authorization`
- **authorize expression**: `Bearer ${token}`

**Tip — cloning providers:** Once your first **Generic HTTP API** dataset provider is working (OAuth + an endpoint), **clone** it in Search to add the next provider. You keep OAuth and the same pattern; then rename the provider, replace endpoints (and dataset binding) per section below. Much faster than configuring each provider from scratch.

---

## Worker groups

Single Leader call listing worker groups and fleets (including Edge fleets).

**Powers:** Edge Node Statistics **Fleet** dropdown (`isFleet==true`); Stream Configuration and Pack Information group pickers.

**Setup**

- **Data → Dataset providers** → **Generic HTTP API** **cribl_worker_groups**
- **Endpoint** `cribl_groups`: datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/master/groups`
- OAuth: **OAuth (all providers)**
- **Data → Datasets** → **cribl_worker_groups**, enable **cribl_groups**

---

## Stream inventory (config)

Config for the selected worker group; URLs use `${worker_group}`.

**Powers:** Routes, pipelines, packs, system inputs, system outputs for the chosen group.

**Setup**

1. **Data → Dataset providers** → **Generic HTTP API** **cribl_stream_inventory**
2. OAuth: **OAuth (all providers)**
3. **Endpoints** (datafield `items`, get; replace `<workspace>-<org>`):

   - **cribl_routes** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/routes`
   - **cribl_pipelines** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/pipelines`
   - **cribl_packs** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/packs`
   - **cribl_inputs** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/system/inputs?includePacks=true`
   - **cribl_outputs** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/system/outputs?includePacks=true`

4. **Data → Datasets** → **cribl_stream_inventory**, enable all five endpoints, add **cribl_stream_inventory** datatype ruleset under processing

---

## Edge Node Statistics

Dashboard: snapshot of Edge fleet workers from **`master/workers`**—fleet-wide totals, top hosts (bars + tables), traffic share by host (pies). Values refresh when you run the search; not a stored time series.

**On the dashboard**

- **Time Range** — Query window for the dataset
- **Fleet** — Edge fleet from **cribl_worker_groups** (* = all); needs **Worker groups** first
- **KPIs** — Totals: in/out events and bytes for the fleet
- **Bar charts** — Top 10 by bytes in and by events in
- **Pies** — Share of in bytes and in events by host
- **Tables** — Top 10 by bytes in and by events in (columns include host, ids, metrics, lastMsgTime); sort falls back to lastMsgTime if metrics are empty

**Setup**

1. **Worker groups** in place (Fleet dropdown).
2. **Data → Dataset providers** → **Generic HTTP API** **cribl_metrics**
3. **Endpoint** `cribl_worker_metrics`: datafield `items`, get — `https://<workspace>-<org>.cribl.cloud/api/v1/master/workers`
4. OAuth: **OAuth (all providers)**
5. **Data → Datasets** → **cribl_worker_metrics**, enable **cribl_worker_metrics**, add **cribl_worker_metrics** datatype ruleset under processing

---

## Pack Details (optional)

Pack-scoped inputs, outputs, routes, pipelines under **`${worker_group}`** and **`${pack}`**.

**Setup**

1. **Data → Dataset providers** → **Generic HTTP API** **cribl_packs**
2. OAuth: **OAuth (all providers)**
3. **Endpoints** (datafield `items`, get):

   - **cribl_packs_inputs** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/inputs`
   - **cribl_packs_outputs** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/outputs`
   - **cribl_packs_routes** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/routes`
   - **cribl_packs_pipelines** — `https://<workspace>-<org>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/pipelines`

4. **Data → Datasets** → **cribl_packs**, enable endpoints, add **cribl_packs** datatype ruleset under processing

---

## Macros and renames

Pack macros live in **default/macros.yml**. If you rename **cribl_worker_groups**, **cribl_stream_inventory**, **cribl_worker_metrics**, or **cribl_packs**, update the matching macro.

---

## Release Notes

- **1.1.9** (2026-02-17) — README refresh: tighter flow, dataset names in What You Get, nested endpoint lists, **default/macros.yml** pointer; Edge Node Statistics + setup; `dataset="..."` macros.
- **1.1.8** (2026-02-17) — Edge Node Statistics: KPIs, bar and pie charts, top-10 tables; line charts removed (JIT API snapshot).
- **1.0.1** (2026-01-27) — Typos and instruction clarifications.
- **0.9.1** (2025-12-19) — Beta release.

---

## Contributing

Reach out to Kelsey Prior (cribl.io) on [Cribl Community Slack](https://cribl-community.slack.com).

## Contact

kprior@cribl.io

## License

[Apache 2.0](https://github.com/criblio/appscope/blob/master/LICENSE)
