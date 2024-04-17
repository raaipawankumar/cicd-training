# Continuous Integration and Deployment

Continuous integration is an application development practice where developers frequently integrate their code changes into a shared repository that triggers an automated build process, which includes compiling the code, running automated tests and checking for potential errors. The main purpose of continuous integration is to detect and address integration issues as quickly as possible.

## Azure Pipeline

Azure Pipeline is a cloud-based CI/CD service. It supports the YAML pipeline that allows us to define or build and release processes as code and provides a more versionable and maintainable approach to a visual designer-based pipeline.

## Azure Pipeline Template

```yaml
# trigger section specifies the conditions that trigger the pipeline to run.
trigger:
  branches:
    include:
    - main
    - development
  paths:
    include:
    - src/*
# parameters section allows us to define input parameters that can be customized when using the template in a pipeline.
parameters:
  vmImage: 'ubuntu-latest' # Used to configure the virtual machine image
  buildConfiguration: 'Release'

# stages section defines the stages of the pipeline. Each stage contains one or more jobs, which represent steps to be executed. Each job specifies the execution environment (vmImage) and a series of steps to be performed.
stages:
- stage: Build
  displayName: 'Build stage'
  jobs:
  - job: BuildJob
    displayName: 'Build job'
    pool:
     #name: 'Name of Agent pool' -> Default name of an Agent Pool is Azure pipelines. To use a self hosted agent, use name of your self hosted agent pool 
      vmImage: ${{ parameters.vmImage }} #vmImage property specifies the Docker container image to be used as the execution environment for the steps.
    steps:
    - script: |
        # Example build steps
        echo 'Building the project...'
        # Replace this with your actual build commands
        # e.g., dotnet build, npm run build, etc.
      displayName: 'Build'

- stage: Test
  displayName: 'Test stage'
  jobs:
  - job: TestJob
    displayName: 'Test job'
    pool:
      vmImage: ${{ parameters.vmImage }}
    steps:
    - script: |
        # Example test steps
        echo 'Running tests...'
        # Replace this with your actual test commands
        # e.g., dotnet test, npm test, pytest, etc.
      displayName: 'Run tests'

# We can exclude Stages if there is only one stage
# We can exclude jobs if there is only one job

```

We can extend a Yaml pipeline from another Yaml pipeline using **extends** key.

```yaml
# File: azure-pipeline.yml

trigger:
  branches:
    include:
    - main

extends:
  template: azure-pipeline-template.yml
  parameters:
    vmImage: 'ubuntu-latest'
    buildConfiguration: 'Release'

```

## Agent Pool

Agent pool is a collection of Agent machines that run pipeline jobs. By default, Azure DevOps comes with 2 Agent pools.

1. Azure Pipelines: Azure Pipelines is the default Microsoft-hosted pool.
2. Default: We can use this to attach our self-hosted agent.

## Pools and Agents

By default, Azure DevOps

Azure DevOps provides two types of agents:

1. Microsoft Hosted Agents: They are managed and maintained by Microsoft. By default, a pipeline uses a Microsoft-hosted pool.
2. Self-Hosted Agents: Self-hosted agents are set up and maintained by us on our infrastructure, This could be an on-premise server or virtual machine. We should use it when we want more control over infrastructure or we have to follow strict security guidelines and compliances.

### Assignment of self-hosted agent to a pipeline
