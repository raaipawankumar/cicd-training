# Continuous Integration and Deployment
## Continuous Integeration
Continuous integration is an application development practice where developers frequently integrate their code changes into a shared code repository that triggers an automated build process, which includes compiling the code, running automated tests and checking for potential errors. The main purpose of continuous integration is to detect and address integration issues as quickly as possible.

## Azure DevOps Pipeline

Azure DevOps Pipeline is a cloud-based CI/CD service. It supports the YAML pipeline that allows us to define or build and release processes as code and provides a more versionable and maintainable approach to a visual designer-based pipeline.

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

## Trigger

Trigger is used to define conditions under which the pipeline should be triggered. There are three types of triggers: Repository, Resource and Schedule.

```yaml
# Disable triggers
trigger: none

# Define triggers on specified branches
trigger:
  - main
  - develop

# Specify trigger with multiple options
trigger:
  batch: true
  branches:
    include:
    - features/*
    exclude:
    - features/experimental/*
  paths:
    exclude:
    - README.md
```

### batch

batch is used when we want to reduce the number of pipeline runs when pushing the changes to the repository very often. If we set batch to true then, when a pipeline is running, the system waits until the run is completed, and then starts another run with all changes that have not yet been built.

### branches

branches configuration is used to specify branch names to include or exclude for triggering a run.

### paths

paths configuration is used to specify file paths to include or exclude for triggering a run.

### tags

tags configuration is used to specify tag names to include or exclude for triggering a run.

## Variables

Variables are used to define values that can be reused throughout the pipeline. Variables can be defined using name-value pairs or using a list of objects with name, value and readonly keys. variables can be used anywhere in the pipeline by enclosing the variable name in $(). We can not use variables in triggers. variables can be scoped to root, stage or job level where defined. There are also system variables that are read-only and provide information about the build, pipeline, agent and system. We can also create secret variables to protect sensitive data such as connection strings and others. We can define them in the portal UI and mark them as secret.

### Variable Group

A variable group is used to store secret values. It supports use of key-vault for storing secrets.

```yaml
# There are two ways to define variables
# 1:
variables:
  var1: 'value-1'
  var2: 'value-2'
# 2:
variables:
- name: var1
  value: value-1
  readonly: true
- group: group-1

```

## Parameters

Parameters provide a way to customize the behavior of the pipeline at runtime. They allow us to define variables that users can specify when triggering the pipeline. They make pipelines configurable and reusable across different scenarios. They enable users to customize pipeline behavior without modifying the pipeline definition, providing consistency and flexibility in CI/CD processes. Parameters can be used anywhere in the pipeline configuration by enclosing the parameter name in $().

```yaml
parameters:
  - name: paramName
    type: string
    default: defaultValue
    displayName: Display Name
    description: Parameter Description
    values:
      - option1
      - option2

```

## Template

A template is used to define reusable content, logic and parameters in the pipeline. There are two types of templates:

1. **Include:** Include allows us to break down our pipeline configuration into smaller and reusable files. We can define common configurations or steps in separate YAML files and then include them in our main pipeline file using the include keyword.
2. **Exclude** Exclude allows us to inherit and override configurations from existing templates. We can define a base template with common configurations and then create specialized templates that extend the base template and override specific configurations as needed.

```yaml
jobs:
- job: Linux
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - template: templates/include-npm-steps.yml  # Template reference
- job: Windows
  pool:
    vmImage: 'windows-latest'
  steps:
  - template: templates/include-npm-steps.yml
```

## Stages

Stages are used to break down the execution of the pipeline into distinct phases, each representing a logical division of work. Stages are defined at the root level using the stages keyword. Stages are defined using an array and each stage is represented as a separate block of YAML.

By default, stages execute sequentially in the order they are defined but we can configure them to run in parallel by specifying dependencies between them.

```yaml
stages:
- stage: Stage1
  displayName: Stage 1
  jobs:
    - job: Job1
      displayName: Job 1
      steps:
      - script: echo "Hello from Job 1"
- stage: Stage2
  displayName: Stage 2
  jobs:
    - job: Job2
      displayName: Job 2
      steps:
      - script: echo "Hello from Job 2"

```

## Jobs

Jobs represent a unit of work that needs to be performed within a stage. It is a collection of jobs where each job is a set of tasks or steps.

```yaml
```yaml
stages:
- stage: Stage1
  displayName: Stage 1
  jobs:
    - job: Job1
      displayName: Job 1
      steps:
      - script: echo "Hello from Job 1"
```

## Steps

Steps are individual actions or commands that are executed as part of a job to accomplish the objectives of the job. Steps are defined within a job in the YAML file.

### Script Step

Script step is used to execute arbitrary commands. It is defined using the **script** keyword. followed by the command to execute. | is used to execute a multiline script.

```yaml
  steps:
  - script: echo "Step 1: Running a script"
  - script: |
      echo "Line 1"
      echo "Line 2"

```

### Task step

The Task step is used to execute a predefined task provided by the pipeline extensions or custom tasks defined within the pipeline. It is defined using the task keyword.

```yaml
- task: SomeTask@1
  inputs:
    parameter1: value1

```

By default, Steps within a job are executed sequentially but we can also configure them to run in parallel using the parallel keyword.

```yaml
- parallel:
  - script: echo "Step 1"
  - script: echo "Step 2"

```
