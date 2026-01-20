## Setup and Running RHOne 2026  Developer Lightspeed for MTA Demo


#### Relevant setup references

**Solution Server**  
https://github.com/konveyor/operator?tab=readme-ov-file#konveyor-operator-installation-on-okdopenshift  

**RH Doc Based Approach for VSCode and OpenShift parts**  
*Configuring and Using Red Hat Developer Lightspeed for MTA*  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index  


**Konveyor git repo for demo Java EE to Quarkus Migration** 
https://github.com/konveyor-ecosystem/coolstore


### Setup Steps and Tips

> Ensure the correct mta/tackle name and namespace is used throughout the installation and configuration for the OpenShift MTA portions. Example snippets in the upstream and Red Hat docs vary in which one they use and this repo may have mixed references in code and files.

#### VSCode Plugin aka MTA Plugin Setup
- For the VSCode Dev LS plugin (Known as the *MTA Plugin* in the vscode repository) configure the plugin after installation  
There are 2 LLM related configurations that need to be set 1/ for the Analysis Server aka GenAI 2/ for the Solution Server
  - https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#configuring-developer-lightspeed-ide-settings_configuring-dev-lightspeed-ide 
  - https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#llm-provider-settings_configuring-llm  
- For demos best to **NOT** have auto-save on for files

- Most likely you will need to use a bigger LLM for both the VSCode plugin usage and Solution Server, current one in use is MaaS hosted  
  *llama-4-scout-17b-16e-w4a16*  

Analysis Server (GenAI) example setting for provider-settings.yaml
*there can be only one &active model* 
this example is for a MaaS hosted vLLM served openai compatible configuration
```

  OpenShift: &active  # only one model can be active
    environment:
      OPENAI_API_KEY: "************************************" # Required
    provider: ChatOpenAI
    args:
      model: llama-4-scout-17b-16e-w4a16 # Required
      configuration:
        baseURL: "https://<serving url>/v1"
```

#### Solution Server Notes and Installation
- Solution Server can be installed and run several ways.  The preferred way is as part of an OpenShift MTA deployment.  Upstream documents and Red Hat Docs should both be reviewed.  
https://github.com/konveyor/operator?tab=readme-ov-file#konveyor-operator-installation-on-okdopenshift  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#tackle-enable-dev-lightspeed_solution-server-configurations  
- Before installing Solution Server: ensure the secret *kai-api-keys* is set correctly for the model and serving approach.  It has to be present before you have the solution server (kai server piece) installed  
*openai compatible example*   
```
oc create secret generic kai-api-keys -n <namesapce for tackle/mta instance> \
--from-literal=OPENAI_API_BASE='<url to model in model server>/v1' \
--from-literal=OPENAI_API_KEY='*******************************'

```
- Install Solution Server by appling changes to an existing tackle env or creating a new one.  Example yaml is in **yaml** directory  
- Keycloak: In most installations keycloak is required and it is normally tied into the MTA installation:  
*link for keycloak access info and mta related roles*  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/installing_the_migration_toolkit_for_applications/index#red-hat-build-of-keycloak_installing-mta-ui  
- example Solution Server settings to go in VSCode settings.json  
*This example is based upon a keycloak secured mta tackle env with kai solution server.*  
several ways to get to settings.json, one way go through ide Settings , Extensions, MTA , Solution Server (has link to settings.json)  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#configuring-solution-server-settings-file_configuring-dev-lightspeed-ide  
  
  ```
      "mta-vscode-extension.solutionServer": {

        "url": "<mta/tackle hub console route url>/hub/services/kai/api",

        "enabled": true,
        "auth": {

           "enabled": true, 
           "insecure": true,
           "realm": "mta"
        },

    }
  ```
- for KAI Solution Server authentication login, the MTA console un/pwd can be used.  
  - the un/pwd is usually a secret *something*-mta-rhsso
- If authentication is not set in tackle/mta during the solution server setup then adjust the above snippet to "enabled": false  


#### Resetting Solution Server i.e. clearing current solution fixes when restarting the demo   
- Alternatively, connecting to the postgres pod and drop all tables with psql, then bounce the pod  
  - *open a terminal for the db pod*
  - psql -U kai
  - *check to see if tables are present*
  - \dt
  - *if so, delete them all*
  - drop table kai_files, kai_hints, kai_incidents, kai_solution_hint_association, kai_solutions, kai_violation_hint_association, kai_violations, solution_after_file_association, solution_before_file_association ; 
  - optional *delete/bounce the kai-db pod*  
  - *delete/bounce kai-api pod* 
  - *quit and restart VSCode*  
    - to re-initialization (create new empty tables) need to hard reset the solution server connection from the MTA Analysis View  
    - to ensure it worked go into MTA Analysis View after restarting and you should NOT see a yellow banner/notification the says "Solution Server Disconnected" 
  
  **(untried approach: Force-deleting the PVC would do it (may need to bounce the pod as well)).**

  #### Tips for Demoing
  - ensure you are using the correct profile in DevLightspeed 
  - turn off auto-save feature for files in vscode
  - possibly reset connection to custom rules everytime you initially start up a demo  
  - ensure LLM configurations  are correct for analysis server (GenAI) and solution server  
  - delete tables for kai-db pod and bounce pod and also kai-api to clean up solution server hints.  
    - Need to also completely quit/kill vscode and restart to reset solution server connection and auto create new tables
  - When using solution server feature, and manually adjusting suggested genai code update,
    - as of 8 Dec 2025 ensure to replace the entire suggested inserted code, even if that means literally copying over some lines with the exact same suggested changes, as solution server seems to get confused and messes up the hints for other follow on files that need the change.







