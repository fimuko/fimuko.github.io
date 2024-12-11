---
title: "Creating a Flexible Azure DevOps YAML Pipeline for Multi-Environment Infrastructure Deployment Using Bicep"
date: 2024-12-11T09:49:12+01:00
# weight: 1
# aliases: ["/first"]
tags: ["devops", "yaml", "bicep"]
categories: ["devops"]
author: "fimuko"
# author: ["Me", "You"] # multiple authors
summary: "a YAML template for Azure DevOps Pipelines that leverages Bicep, Microsoft's infrastructure-as-code (IaC) language" # this shows up on the list
description: "a YAML template for Azure DevOps Pipelines that leverages Bicep, Microsoft's infrastructure-as-code (IaC) language" # this shows up on the single page
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
#canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: false # keep this false as if true it is not showing H1
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link
---

Deploying infrastructure and applications across multiple environments can be a complex and time-consuming process. To simplify and streamline this, I developed a YAML template for Azure DevOps Pipelines that leverages Bicep, Microsoft's infrastructure-as-code (IaC) language. This template provides a user-friendly way to deploy applications across different environments (DEV, TST, UAT, PRD) with a manual pipeline trigger that includes selectable checkboxes for applications and environments.

This template is designed for scenarios where environments consist of multiple applications, each managed with a dedicated Bicep file. The deployments are orchestrated through a single Azure DevOps pipeline (using one YAML file). With this setup, Azure DevOps users (i.e. operators who trigger the deployment of applications) can dynamically select which parts of the environment (i.e. specific applications within specific environments) they wish to deploy during each pipeline run. This approach balances simplicity and control, ensuring deployments are both flexible and streamlined without the need for multiple pipelines or redundant configurations.

In this blog post, I'll walk you through the challenges this template addresses, how it works, and how you can use it in your projects.


## The Problem

Managing deployments across multiple environments typically involves:
- Maintaining separate pipelines for each environment.
- Manual modifications to specify which applications to deploy.
- Risk of errors due to repetitive and manual processes.

These challenges grow as the number of applications and environments increases, leading to inefficiencies and deployment delays. What was missing was a simple yet powerful mechanism to:
- Dynamically select applications and environments during pipeline execution.
- Automate infrastructure deployment using a repeatable template.


## The Solution

To address these challenges, I created a YAML pipeline template for Azure DevOps that:
- Integrates with Bicep for seamless infrastructure-as-code deployment.
- Allows users to manually select applications and environments through checkboxes when triggering the pipeline.
- Supports multi-environment setups with reusable and scalable configurations.

This solution brings flexibility, scalability, and simplicity to multi-environment deployments, making it easier to manage complex infrastructure setups.


## How It Works

### Pipeline Overview

The pipeline consists of three YAML files:
1. **`main.yaml`**: The entry point for the pipeline. It defines the parameters for user input and passes them to the stages.
2. **`stages.yaml`**: Defines the stages for deploying to the selected environments.
3. **`env-template.yaml`**: Contains the logic for deploying selected applications to a single environment.

### `main.yaml`

The main YAML file defines parameters that allow users to select the applications and environments to deploy. These (boolean) parameters are displayed as checkboxes during pipeline execution. Here's the code:


```yaml
# main.yaml
trigger: none

pool:
  vmImage: ubuntu-latest

parameters:
    # parameters for selecting apps we want to deploy
  - name: deployApp1
    displayName: "Deploy App1"
    type: boolean
    default: false
  - name: deployApp2
    displayName: "Deploy App2"
    type: boolean
    default: false
    # parameters for selecting envs we want to deploy
  - name: deployDEV
    displayName: Deploy to DEV environment
    type: boolean
    default: false
  - name: deployTST
    displayName: Deploy to TST environment
    type: boolean
    default: false
  - name: deployUAT
    displayName: Deploy to UAT environment
    type: boolean
    default: false
  - name: deployPRD
    displayName: Deploy to PRD environment
    type: boolean
    default: false
    
stages:
  - template: stages.yaml
    parameters:
      deployApp1: ${{ parameters.deployApp1 }}
      deployApp2: ${{ parameters.deployApp2 }}
      deployDEV: ${{parameters.deployDEV}}
      deployTST: ${{parameters.deployTST}}
      deployUAT: ${{parameters.deployUAT}}
      deployPRD: ${{parameters.deployPRD}}
```

All this simple file does is:
- Defines parameters for application and environment selection.
- Passes these parameters to the stages defined in `stages.yaml`.

### `stages.yaml`

This file defines the logic for deploying applications across selected environments. It uses an array of objects to manage environment-specific configurations. Here's the code:

```yaml
# stages.yaml
parameters:
    # parameters passed from main.yaml
  - name: deployApp1
    displayName: "Deploy App1"
    type: boolean
    default: false
  - name: deployApp2
    displayName: "Deploy App2"
    type: boolean
    default: false
  - name: deployDEV
    displayName: Deploy to DEV environment
    type: boolean
    default: false
  - name: deployTST
    displayName: Deploy to TST environment
    type: boolean
    default: false
  - name: deployUAT
    displayName: Deploy to UAT environment
    type: boolean
    default: false
  - name: deployPRD
    displayName: Deploy to PRD environment
    type: boolean
    default: false
    # now a bunch of additional parameters with default values
    # these ideally should be something like a constant or a variables, 
    # however in yaml variables can only be strings,
    # and in some cases we actually need arrays / dictionaries to iterate over their elements, 
    # hence we put what we need into parameters of type object
  - name: environments
    type: object
    default:
      - name: DEV
        paramsFileApp1: 'deployment/parameters/app1.main.dev.bicepparam'
        paramsFileApp2: 'deployment/parameters/app2.main.dev.bicepparam'
        connectionApp1: '<subscr>' # subscription name hosting service connection to Azure DevOps 
        connectionApp2: '<subscr>' # subscription name hosting service connection to Azure DevOps
      - name: TST
        paramsFileApp1: 'deployment/parameters/app1.main.tst.bicepparam'
        paramsFileApp2: 'deployment/parameters/app2.main.tst.bicepparam'
        connectionApp1: '<subscr>' # subscription name hosting service connection to Azure DevOps 
        connectionApp2: '<subscr>' # subscription name hosting service connection to Azure DevOps
      - name: UAT
        paramsFileApp1: 'deployment/parameters/app1.main.uat.bicepparam'
        paramsFileApp2: 'deployment/parameters/app2.main.uat.bicepparam'
        connectionApp1: '<subscr>' # subscription name hosting service connection to Azure DevOps 
        connectionApp2: '<subscr>' # subscription name hosting service connection to Azure DevOps
      - name: PRD
        paramsFileApp1: 'deployment/parameters/app1.main.prd.bicepparam'
        paramsFileApp2: 'deployment/parameters/app2.main.prd.bicepparam'
        connectionApp1: '<subscr>' # subscription name hosting service connection to Azure DevOps 
        connectionApp2: '<subscr>' # subscription name hosting service connection to Azure DevOps
    # bicep files are actually fixed / independent of environment, 
    # so those do not need to fall under the object above, they can just be strings with a default value
  - name: bicepFileApp1
    type: string
    default: 'app1.main.bicep'
  - name: bicepFileApp2
    type: string
    default: 'app2.main.bicep'  

stages:
  - ${{ each environment in parameters.environments }}:
      - stage: DeployAppsTo${{ environment.name }}
        displayName: "Deploy Apps to ${{ environment.name }}"
        # below we dynamically construct the name of the parameter we are interested in, depending on the environment,
        # and we grab its value into a new variable 'deployEnvFlagValue`
        # for example if the environment='DEV', we grab the value (true/false) of the deployDEV parameter,
        # and we capture is as a deployEnvFlag variable
        variables:
          deployEnvFlagValue: ${{ parameters[format('deploy{0}', environment.name)] }}
        # now we can use this new variable in the condition used for conditional deployment
        condition: and(not(canceled()), eq(variables['deployEnvFlagValue'], 'true'))
        jobs:
          - template: env-template.yaml
            parameters:
              environmentName: ${{ environment.name }}
              deployApp1: ${{ parameters.deployApp1 }}
              deployApp2: ${{ parameters.deployApp2 }}
              connectionApp1: ${{ environment.connectionApp1 }}
              connectionApp2:  ${{ environment.connectionApp2 }}
              bicepFileApp1: ${{ parameters.bicepFileApp1 }}
              bicepFileApp2: ${{ parameters.bicepFileApp2 }}
              paramsFileApp1: ${{ environment.paramsFileApp1 }}
              paramsFileApp2: ${{ environment.paramsFileApp2 }}
```

Key points:
- Environment-specific configurations are stored in an array of objects.
- The pipeline iterates over selected environments.

### `env-template.yaml`

This file handles the actual deployment logic for each environment. It will iterate over each application we have in our architecture. Here's the code:

```yaml
# env-template.yaml
parameters:
  - name: environmentName
    type: string
  - name: deployApp1
    type: boolean
  - name: deployApp2
    type: boolean
  - name: paramsFileApp1
    type: string
  - name: paramsFileApp2
    type: string
  - name: connectionApp1
    type: string
  - name: connectionApp2
    type: string
  - name: bicepFileApp1
    type: string
  - name: bicepFileApp2
    type: string    

jobs:
  - job: SetVariables
    displayName: "set variables"
    # this string variable is created solely for the purpose of later splitting it on comma
    # so that we can iterate over the split result
    # we do this because it is not possible to have object / array type of variable here
    # so we had to get creative if we wanted an array
    variables:
      appNames: 'App1,App2'
    steps:
      - script: echo "setting variable for iteration - list of apps to deploy"

  - ${{ each appName in split(variables.appNames, ',')}}:
    - job: Deploy${{ appName }}${{ parameters.environmentName }}
      displayName: "Deploy ${{ appName }} to ${{ parameters.environmentName }}"
      variables:
        location: '<location string>'
        # in a simiilar way to the stages.yaml, here we are grabbing either deployApp1 or deployApp2 value
        # and we are capturing it in a variable that we can use in a condition:
        deployAppFlagValue: ${{ parameters[format('deploy{0}', appName)] }}
        # the same goes for bicepFileApp1, bicepFileApp2 files:
        bicepFile: ${{ parameters[format('bicepFile{0}', appName)] }}
        # and so on:
        paramsFile: ${{ parameters[format('paramsFile{0}', appName)] }}
        connection: ${{ parameters[format('connection{0}', appName)] }}
      condition: and(not(canceled()), eq(variables['deployAppFlagValue'], 'true'))
      steps:
        - task: AzureCLI@2
          displayName: 'Validate bicep template'
          inputs:
            azureSubscription: '${{ variables.connection }}'
            scriptLocation: inlineScript
            scriptType: bash
            inlineScript: |                        
              az deployment sub what-if --location ${{variables.location}} \
                --name 'iac_${{lower(parameters.environmentName)}}_'$(date +"%Y-%m-%d_%H-%M-%S_%Z") \
                --template-file ${{ variables.bicepFile }} \
                --parameters ${{variables.paramsFile}}  
        
        - task: AzureCLI@2
          displayName: 'Deploy bicep template'
          inputs:
            azureSubscription: '${{ variables.connection }}'
            scriptLocation: inlineScript
            scriptType: bash
            inlineScript: |                
              az deployment sub create --location ${{variables.location}} \
                --name 'iac_${{lower(parameters.environmentName)}}_'$(date +"%Y-%m-%d_%H-%M-%S_%Z") \
                --template-file ${{ variables.bicepFile }} \
                --parameters ${{variables.paramsFile}}
```

This file:
- Iterates over applications and deploys them to the selected environment.
- Uses Azure CLI tasks to validate and deploy Bicep templates.
- Handles YAML limitations with creative use of variables.


## Benefits of the Approach

This approach provides several advantages:
- **Flexibility**: Users can choose specific applications and environments to deploy, reducing unnecessary deployments.
- **Scalability**: Easily extend the template to include additional applications or environments.
- **Efficiency**: Eliminates repetitive YAML configurations, saving time and reducing errors.
- **Infrastructure as Code**: Ensures deployments are consistent and reproducible.


## Conclusion

This Azure DevOps YAML template simplifies multi-environment infrastructure deployments by giving users granular control over deployment. It ensures YAML is succinct and efficient, easy to analyse and troubleshoot.
