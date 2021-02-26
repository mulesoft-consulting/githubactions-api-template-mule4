# template-api

## DEFINITIONS

| Terme           | description          |
|-----------------|----------------------|
| Branch Name     | is a Git branch name |
| Execution Environment | Each mule app takes a certain amount of initial parameters (e.g credentials or connection keys) and vary from an environment to another. these paremters are store in mule's internal vault and are retrieved depending on the provided environment name|
| Anypoint Environment| The cloudhub environment where applications are running |
## TEMPLATE

### Description

This project is a simple template that integrates some of MuleSoft best practices. It provides a set of standard minimalistic tools that should help you start your project quickly/efficiently.

  - **Plugins**
      - mule-maven-plugin: enriched with a configuration that goes along with the Jenkinsfile (pipeline).
      - maven-release-plugin: maven release management tool see the **Realse** section for more information on how to use it
  - **Structure**
      - Separation between interface and implementation
      - Global configuration file
      - Resources
          - property files: 
            - common property file containing environment variable common to all environments (sandbox to production) like http port, basePath etc ...
            - jsonlogger property files, used to configure the JSON Logger plugin
            - a property file for each environment DEV/QA/UAT. You can add as many as you want (just don't forget to update the jenkinsfile)
  - **Dependencies** Your organisation should add the following dependencies to exchange.
      - json-logger
      - common-error-handler
  - **CI/CD**
      - github workflow: a generic premade pipeline. The pipeline uses a number of secrets that should be setup in github. Checkout the Secrets section below. 

### INSTANTIATE

Make sure the `instantiate.sh` file is executable
```bash
$ chmod +x instantiate.sh
```

Below is how to use the instantiation script:

```bash
$ ./instantiate.sh [api-name] [group-id] [group-name] [api-repo-url] [anypoint-host] [region]
```

Where the options are: 

| parameters      | description    | example       | default    |
|-----------------|----------------|---------------|------------|
|`api-name`       |**required** the name of the new project| my-project-name | |
|`group-id`       |The business group id (cloudhub). | 0aefazeg0aazera2 |  |
|`group-name`     |The business group name (cloudhub)| MyBusinessGroup  | |
|`api-repo-url`   |The repository url (ssh or http) of the api| git@github:user/repository.git | |
|`anypoint-host`  | The anypoint host | anypoint.mulesoft.com | anypoint.mulesoft.com |
|`region`         | The anypoint region | eu-central-1 | us-east-1 |


> **Note:** those parameters should be given in the **right order**. Only the `api-name` is required. 

Example: 

```bash
$ ./instantiate.sh  \
      my-api-name \
      07ac71-97cb-46c0-ad91-105eb78e8 \
      myBusinessGroupName \
      git@github:user/repository.git \  
      eu1.anypoint.mulesoft.com \
      eu-central-1
```

The new project will be created in the same parent folder as the template folder. 


### Pipeline: Github Actions

This project contains a templated github actions pipeline. The pipeline is triggered with the following events: 
  * On push
  * Manually executed

The pipeline assumes that a branch name is equivalent to an execution environment. In fact, the execution environment's name should be the uppercase of the branch name. Example: 
  * The branch `dev` is equivalent to the execution environment `DEV`.
  * The branch `qa` is equivalent to the execution environment `QA`

Some environment specific variables should be set if you are planning to use other branches and environments than `dev` and `qa`. checkout the following snippet: 

```yaml
# DEV branch specific configuration
- if: github.ref == 'refs/heads/dev'
  name: Environment DEV configuration
  run: |
    echo 'export ENV="DEV"' > ~/.bashrc
    echo 'export ANYPOINT_ENV="Development"' > ~/.bashrc

# QA branch specific configuration 
- if: github.ref == 'refs/heads/qa'
  name: Environment QA configuration
  run: |
    echo 'export ENV="QA"' > ~/.bashrc
    echo 'export ANYPOINT_ENV="Quality"' > ~/.bashrc
```
Therefore, for a given branch name, the following parameters should be specified
- **ENV** is the execution environemnt
- **ANYPOINT_ENV** is cloudhub's environment name 

The pipeline uses secrets that are detailed in the next section.

### Setup Secrets

The project comes with a pipeline based on **github actions**. It is mandatory to setup secrets at the github organization or repository level.

The following is a list of secrets mandatory to make the pipeline work:

**N.B:** When a secret name contains `[ENV_NAME]` it means that it is a dynamic name (where the `ENV_NAME` is actually the uppercase of the branch name) that is calculated depending on the branch name.

| parameters      | description    | example       |
|-----------------|----------------|---------------|
|`ANYPOINT_APP_CLIENT_ID`                 |**required** The Anypoint platform's - connected app - client id     | azervzn-azer12-a12-azzaeraz |
|`ANYPOINT_APP_CLIENT_SECRET`             |**required** The Anypoint platform's - connected app - client secret | svdf-aaazvs-sdvrffbg-zrg    |
|`[ENV_NAME]__ANYPOINT_ENV_CLIENT_ID`     |**required** The Anypoint platform's environment client id           | azervzn-azer12-a12-azzaeraz |
|`[ENV_NAME]__ANYPOINT_ENV_CLIENT_SECRET` |**required** The Anypoint platform's environment client secret       | svdf-aaazvs-sdvrffbg-zrg    |
|`[ENV_NAME]__MULE_VAULT_KEY`             |**required** The mule application's vault key to decrypt mule's secured properties| 0123456701234567|
|`NEXUS_EE_PWD`                           |**required** The nexus entreprise repository password               |  azertat   |
|`NEXUS_EE_USERNAME`                      |**required** The nexus entreprise repository username               | mule-nexus-user |


## RELEASE

In order to prepare a release, execute the following : 

```bash
mvn release:clean release:prepare 
```

You will be prompted to answer a few questions about the release: 

  1) **What is the release version for ...**: the name of the release. should be something like `X.X.X`. Hit enter to accept the default
  2) **What is the SCM release tag or label for ...**: the name of the tag. The default nomenclature is `{project_name}-{version}`. Use the nomenclature `v{version}`, for example `v1.0.5`. Then hit enter.
  3) **What is the new development version for ...**: the development name or the upcoming version you will be working on (the SNAPSHOT version). It should end with `SNAPSHOT`. The pluging automatically increases the patch number. Hit enter. 

You will notice the following temporary files have been created in the root folder : 

  - **release.properties:** contains the release configuration (release tag etc ...)
  - **pom.xml.releaseBackup:** the old content of the `pom.xml` file with the previous release

After you've reviewed the release properties, execute the following to perform the release:

```bash
mvn release:perform 
```
