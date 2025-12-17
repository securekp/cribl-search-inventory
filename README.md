# Cribl Inventory 
----

This pack is meant to help you see your Cribl infrastructure and configurations easier. It displays all worker groups, routes, pipelines, and packs in a single dashboard.

## About this Pack

Here are some of the benefits this Pack provides:
* Leverages the HTTP API Dataset provider within Cribl Search to pull data from the Cribl Stream API
* Allows for new administrators to easily see what has been configured so far

## Deployment

This pack does require some custom configuration. Follow the below instructions to properly set up the dataset provider. 

### Configure Dataset Provider for Worker Groups

- Navigate to Data -> Dataset providers.
- Create a new Dataset provider the type of Generic HTTP API with a name of cribl_worker_groups
- Add an Endpoint with the following information:
  - name: cribl_groups
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/master/groups
- Go to the Authorization tab
- Select oAuth. This will require that you create API credentials with owner permissions. You can reference the following doc on creating the credentials: https://docs.cribl.io/api#criblcloud
- Fill in the following parameters required for oAuth:
  - Login url: https://login.cribl.cloud/oauth/token
  - client secret parameter: client_secret
  - client secret value: secret for the api credential you created in cloud
- Under extra auth parameters, fill in the following info for the following 3 parameters (case sensitivity matters!):
  - name: audience / value: https://api.cribl.cloud
  - name: client_id / value: client_id value from api key you created in cloud
  - name: grant_type / value: client_credentials
- The remaining configurations after parameters need the following values. (case sensitivity matters!)
  - token_attribute: access_token
  - Authorization header: Authorization
  - authorize expression: Bearer ${token}

### Configure Dataset Provider for Configuration Items
- Navigate to Data -> Dataset providers.
- Create a new Dataset provider the type of Generic HTTP API with a name of cribl_stream_inventory
- Add 5 Endpoints with the following information:
  - name: cribl_routes
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/routes
  - name: cribl_pipelines
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/pipelines
  - name: cribl_packs
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/packs
  - name: cribl_inputs
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/system/inputs?includePacks=true
  - name: cribl_outputs
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/system/outputs?includePacks=true
- Go to the Authorization tab
- Select oAuth. This will require that you create API credentials with owner permissions. You can reference the following doc on creating the credentials: https://docs.cribl.io/api#criblcloud
- Fill in the following parameters required for oAuth:
  - Login url: https://login.cribl.cloud/oauth/token
  - client secret parameter: client_secret
  - client secret value: secret for the api credential you created in cloud
- Under extra auth parameters, fill in the following info for the following 3 parameters (case sensitivity matters!):
  - name: audience / value: https://api.cribl.cloud
  - name: client_id / value: client_id value from api key you created in cloud
  - name: grant_type / value: client_credentials
- The remaining configurations after parameters need the following values. (case sensitivity matters!)
  - token_attribute: access_token
  - Authorization header: Authorization
  - authorize expression: Bearer ${token}

### Configure Cribl Worker Groups Dataset
- Navigate to datasets and create a new dataset called cribl_worker_groups with the following configurations. Note: If you name this dataset something other than cribl_worker_groups, you will need to update the cribl_worker_groups macro to reflect the name you have chosen.
- Dataset provider: cribl_worker_groups
- Under enabled endpoints add cribl_groups endpoints
- Save the dataset.

### Configure Cribl Inventory Dataset
- Navigate to datasets and create a new dataset called cribl_stream_inventory with the following configurations. Note: If you name this dataset something other than cribl_stream_inventory, you will need to update the cribl_stream_inventory macro to reflect the name you have chosen.
- Dataset provider: cribl_stream_inventory
- Under enabled endpoints add cribl_routes, cribl_pipelines, cribl_packs, cribl_inputs, and cribl_outputs endpoints
- Under Processing, add the datatype rulset from the pack: cribl_stream_inventory
- Save the dataset.


### OPTIONAL: Configure Dataset Provider and Dataset for Packs Information
If you want to have an additional dashboard populated with information from individual packs, set up this dataset provider and dataset. This will display pack inputs/outputs/routes/pipelines dependent on which pack you select from a dropdown.
- Navigate to Data -> Dataset providers.
- Create a new Dataset provider the type of Generic HTTP API with a name of cribl_packs
- Add an Endpoint with the following information:
  - name: cribl_packs_inputs
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/inputs
  - name: cribl_packs_outputs
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/system/outputs
  - name: cribl_packs_routes
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/routes
  - name: cribl_packs_pipelines
    - datafield: items
    - method: get
    - url: https://<workspace>-<your org name>.cribl.cloud/api/v1/m/${worker_group}/p/${pack}/pipelines
- Go to the Authorization tab
- Select oAuth. This will require that you create API credentials with owner permissions. You can reference the following doc on creating the credentials: https://docs.cribl.io/api#criblcloud
- Fill in the following parameters required for oAuth:
  - Login url: https://login.cribl.cloud/oauth/token
  - client secret parameter: client_secret
  - client secret value: secret for the api credential you created in cloud
- Under extra auth parameters, fill in the following info for the following 3 parameters (case sensitivity matters!):
  - name: audience / value: https://api.cribl.cloud
  - name: client_id / value: client_id value from api key you created in cloud
  - name: grant_type / value: client_credentials
- The remaining configurations after parameters need the following values. (case sensitivity matters!)
  - token_attribute: access_token
  - Authorization header: Authorization
  - authorize expression: Bearer ${token}
- Navigate to datasets and create a new dataset called cribl_packs with the following configurations. Note: If you name this dataset something other than cribl_packs, you will need to update the cribl_packs macro to reflect the name you have chosen.
  - Dataset provider: cribl_packs
  - Under enabled endpoints add cribl_packs_inputs, cribl_packs_outputs, cribl_packs_routes, cribl_packs_pipelines endpoints
  - Save the dataset.

## Release Notes

### Version 0.9.0 - 2025-12-17

Beta Release

## Contributing to the Pack

To contribute to this Pack, or to report any issues or enhancement requests, please connect with Kelsey Prior (cribl.io) on [Cribl Community Slack](https://cribl-community.slack.com).

## Contact
To contact us please email <kprior@cribl.io>.

## License
This Pack uses the following license: [`Apache 2.0`](https://github.com/criblio/appscope/blob/master/LICENSE).