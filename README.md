### Setup and Running RHOne 2026  Developer Lightspeed for MTA Demo


#### Relevant setup references

**Solution Server**  
https://github.com/konveyor/operator?tab=readme-ov-file#konveyor-operator-installation-on-okdopenshift  

**RH Doc Based Approach for VSCode and OpenShift parts**  
*Configuring and Using Red Hat Developer Lightspeed for MTA*  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index  


**Konveyor git repo for demo Java EE to Quarkus Migration** 
https://github.com/konveyor-ecosystem/coolstore


#### Setup Steps and Tips

> Ensure the correct mta/tackle name and namespace is used throughout the installationa and configuration for the OpenShift MTA portions. Example snippets in the upstream and Red Hat docs vary in which one they use and this repo may have mixed references in code and files.

- For the VSCode Dev LS plugin (Known as the *MTA Plugin* in the vscode repository) configure the plugin after installation  
  - https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#configuring-developer-lightspeed-ide-settings_configuring-dev-lightspeed-ide 
  - https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#llm-provider-settings_configuring-llm  
- For demos best to **NOT** have auto-save on for files

- Most likely you will need to use a bigger LLM for both the VSCode plugin usage and Solution Server, current one in use is MaaS hosted  
  *llama-4-scout-17b-16e-w4a16*  
- Solution Server can be installed and run several ways.  The preferred way is as part of an OpenShift MTA deployment.  Upstream documents and Red Hat Docs should both be reviewed.  
https://github.com/konveyor/operator?tab=readme-ov-file#konveyor-operator-installation-on-okdopenshift  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#tackle-enable-dev-lightspeed_solution-server-configurations  
- ensure the secret *kai-api-keys* is set correctly for the model and serving approach.  It has to be present before you have the solution server (kai server piece) installed
- Install Solution Server by appling changes to an existing tackle env or creating a new one.  Example yaml is in **yaml** directory  
- Keycloak: In most installations keycloak is required and it is normally tied into the MTA installation:  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/installing_the_migration_toolkit_for_applications/index#red-hat-build-of-keycloak_installing-mta-ui  
- example Solution Server settings to go in VSCode settings.json  
*This example is based upon a keycloak secured mta tackle env with kai solution server.*  
several ways to get to settings.json, one way go through ide Settings , Extensions, MTA , Solution Server (has link to settings.json)  
https://docs.redhat.com/en/documentation/migration_toolkit_for_applications/8.0/html-single/configuring_and_using_red_hat_developer_lightspeed_for_mta/index#configuring-solution-server-settings-file_configuring-dev-lightspeed-ide  
  
  ```
      "mta-vscode-extension.solutionServer": {

        "url": "https://mta-openshift-mta.apps.cluster-jd6wh.dynamic.redhatworkshops.io/hub/services/kai/api",

        "enabled": true,
        "auth": {

           "enabled": true, 
           "insecure": true,
           "realm": "mta"
        },

    }
  ```
- for KAI Solution Server authentication login, the MTA console un/pwd can be used.  


#### Resetting Solution Server i.e. clearing current solution fixes when restarting the demo  
- Force-deleting the PVC would do it (may need to bounce the pod as well).  
- Alternatively, connecting to the postgres pod and drop all tables with psql, then bounce the pod  
  - *open a terminal for the db pod*
  - psql -U kai
  - *check to see if tables are present*
  - \dt
  - *if so, delete them all*
  - drop table kai_files, kai_hints, kai_incidents, kai_solution_hint_association, kai_solutions, kai_violation_hint_association, kai_violations, solution_after_file_association, solution_before_file_association  
  - *delete/bounce the kai-db pod*






