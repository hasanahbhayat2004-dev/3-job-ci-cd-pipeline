# 3-Job CI/CD Pipeline

## Table of Contents

- [3-Job CI/CD Pipeline](#3-job-cicd-pipeline)
  - [Table of Contents](#table-of-contents)
  - [1. Overview](#1-overview)
    - [Objectives](#objectives)
  - [2. Pipeline Architecture](#2-pipeline-architecture)
  - [3. Prerequisites](#3-prerequisites)
    - [Repository Structure](#repository-structure)
  - [4. Job 1 — Test](#4-job-1--test)
    - [Purpose](#purpose)
    - [Process](#process)
    - [Why testing comes first](#why-testing-comes-first)
    - [Expected Outcome](#expected-outcome)
  - [5. Job 2 — Merge](#5-job-2--merge)
    - [Purpose](#purpose-1)
    - [Process](#process-1)
    - [Git Branching](#git-branching)
  - [6. Job 3 — Deploy](#6-job-3--deploy)
    - [Purpose](#purpose-2)
    - [Process](#process-2)
  - [7. GitHub Webhook](#7-github-webhook)
    - [Purpose](#purpose-3)
    - [Process](#process-3)
  - [8. AWS EC2 Deployment](#8-aws-ec2-deployment)
    - [Deployment Flow](#deployment-flow)
  - [9. Pipeline Flow](#9-pipeline-flow)
    - [End-to-End Workflow](#end-to-end-workflow)
  - [10. Testing and Results](#10-testing-and-results)
    - [Testing Process](#testing-process)
  - [12. Benefits of the CI/CD Pipeline](#12-benefits-of-the-cicd-pipeline)
    - [Benefits Observed](#benefits-observed)
    - [Benefits for an Organisation](#benefits-for-an-organisation)

## 1. Overview

This document explains the implementation of a 3-job Continuous Integration and Continuous Deployment (CI/CD) pipeline using Jenkins, GitHub and AWS EC2.

The pipeline automates the process of taking changes made to the application, testing them, merging approved changes into the main branch, and deploying the updated application to an AWS EC2 instance.

The pipeline consists of three Jenkins jobs:

- **Job 1 — Test:** Retrieves the application code and runs automated tests.
- **Job 2 — Merge:** Runs after successful testing and merges the development branch into the main branch.
- **Job 3 — Deploy:** Transfers the tested application from Jenkins to an AWS EC2 instance and starts/restarts the application.

A GitHub webhook is used to notify Jenkins when changes are pushed to the repository, allowing the pipeline to be triggered automatically.

### Objectives

The main objectives of this pipeline are to:

- Automate application testing.
- Ensure only tested code is merged into the main branch.
- Automate deployment to an AWS EC2 instance.
- Reduce manual steps between development and deployment.
- Demonstrate the principles of Continuous Integration and Continuous Deployment.
## 2. Pipeline Architecture

The CI/CD pipeline connects GitHub, Jenkins and AWS EC2. Each stage has a specific responsibility, allowing code to be tested, merged and deployed in a controlled sequence.

```text
Developer
    │
    │ Pushes changes
    ▼
GitHub Repository
    │
    │ GitHub Webhook
    ▼
Jenkins Job 1
    │
    │ Run tests
    │
    ├── Tests fail ──► Pipeline stops
    │
    ▼
Jenkins Job 2
    │
    │ Merge dev → main
    ▼
Jenkins Job 3
    │
    │ Copy tested application
    │ via SSH/SCP
    ▼
AWS EC2 Instance
    │
    │ Start/restart application
    ▼
Running Application
```
## 3. Prerequisites

The following tools and services were required to build and run the CI/CD pipeline.

| Tool / Service | Purpose |
|---|---|
| Git | Used for version control and managing branches locally. |
| GitHub | Used to host the application repository and manage the development and main branches. |
| Jenkins | Used to automate testing, merging and deployment. |
| GitHub Webhook | Used to automatically notify Jenkins when changes are pushed. |
| AWS EC2 | Used to host the deployed application. |
| SSH | Used by Jenkins to securely connect to the EC2 instance. |
| SCP / Rsync | Used to transfer the tested application from Jenkins to the EC2 instance. |
| Node.js / npm | Used to install dependencies and run the application on the EC2 instance. |

### Repository Structure

The application was stored in a GitHub repository with separate development and main branches.

The development branch was used for making changes and testing new functionality. Once the tests passed successfully, Jenkins merged the changes into the main branch.

This provided a simple workflow for controlling which code was considered ready for deployment.
## 4. Job 1 — Test

The first Jenkins job is responsible for testing the application before any changes are merged into the main branch or deployed to AWS.

### Purpose

The purpose of Job 1 is to provide an automated quality check. If the tests fail, the pipeline should stop so that untested or broken code is not merged or deployed.

### Process

The job performs the following steps:

1. Jenkins retrieves the application code from the GitHub repository.
2. The required dependencies are installed.
3. The application's automated tests are executed.
4. Jenkins reports whether the tests passed or failed.
5. If the tests pass, the pipeline can continue to Job 2.
6. If the tests fail, the pipeline stops and the code is not merged or deployed.

### Why testing comes first

Testing is placed at the beginning of the pipeline so that problems can be identified before the code reaches the main branch or production environment.

This follows the Continuous Integration principle of integrating and testing changes frequently.

### Expected Outcome

A successful Job 1 execution should show a successful Jenkins build and passing tests.

If the tests fail, the build should be marked as failed and the subsequent jobs should not proceed.

## 5. Job 2 — Merge

The second Jenkins job is responsible for merging tested changes from the development branch into the main branch.

### Purpose

The purpose of Job 2 is to ensure that only code which has successfully passed the testing stage is merged into the main branch.

This provides an additional level of control between development and deployment.

### Process

The job performs the following steps:

1. Job 2 is triggered after Job 1 completes successfully.
2. Jenkins checks that the tests from Job 1 have passed.
3. The changes from the development branch are merged into the main branch.
4. Jenkins pushes the updated main branch back to the GitHub repository.
5. If the merge is successful, the pipeline can continue to Job 3.

If Job 1 fails, Job 2 should not run because the code has not passed the testing stage.

### Git Branching

The pipeline uses two main branches:

- **dev** — used for developing and testing changes.
- **main** — contains the tested code that is ready for deployment.

The workflow can therefore be represented as:

```text
dev branch
    │
    │ Changes pushed
    ▼
Job 1 — Test
    │
    │ Tests pass
    ▼
Job 2 — Merge
    │
    │ dev → main
    ▼
main branch
```
## 6. Job 3 — Deploy

The third Jenkins job is responsible for deploying the tested and merged application to an AWS EC2 instance.

### Purpose

The purpose of Job 3 is to automate the deployment of the application after the code has successfully passed testing and been merged into the main branch.

This removes the need to manually transfer the application and start it on the EC2 instance.

### Process

The job performs the following steps:

1. Job 3 runs after Job 2 completes successfully.
2. Jenkins prepares the updated application files.
3. Jenkins connects securely to the AWS EC2 instance using SSH.
4. The updated application files are transferred from Jenkins to the EC2 instance.
5. Dependencies are installed or updated using npm.
6. The application is started or restarted on the EC2 instance.
7. The updated application becomes available through the EC2 instance.

The deployment flow is:

```text
Job 2 — Merge
      │
      │ Successful merge
      ▼
Job 3 — Deploy
      │
      │ SSH connection
      ▼
AWS EC2
      │
      │ Transfer application
      ▼
Install dependencies
      │
      ▼
Start / restart application
      │
      ▼
Updated application
```
## 7. GitHub Webhook

A GitHub webhook was used to automatically notify Jenkins when changes were pushed to the GitHub repository.

### Purpose

Without a webhook, Jenkins would need to regularly check GitHub to see whether new changes had been made.

The webhook allows GitHub to send an event to Jenkins as soon as a relevant change occurs.

This helps make the CI/CD pipeline more automated and responsive.

### Process

The workflow is:

```text
Developer
    │
    │ git push
    ▼
GitHub
    │
    │ Webhook notification
    ▼
Jenkins
    │
    ▼
Job 1 — Test
```
## 8. AWS EC2 Deployment

AWS EC2 was used as the cloud environment for hosting the application.

The EC2 instance provides the compute environment required to run the application after Jenkins has completed the CI/CD process.

### Deployment Flow

The deployment process can be summarised as:

```text
GitHub
   │
   ▼
Jenkins
   │
   │ Tested & merged code
   ▼
SSH connection
   │
   ▼
AWS EC2 Instance
   │
   ├── Application files transferred
   ├── Dependencies installed
   └── Application started
   │
   ▼
Running Application
```
## 9. Pipeline Flow

The complete CI/CD pipeline follows a controlled sequence from code development through to deployment.

### End-to-End Workflow

```text
┌──────────────┐
│  Developer   │
│              │
│ Writes code  │
└──────┬───────┘
       │
       │ git push
       ▼
┌──────────────┐
│    GitHub    │
│              │
│  dev branch  │
└──────┬───────┘
       │
       │ Webhook
       ▼
┌──────────────┐
│ Jenkins Job 1│
│              │
│    TEST      │
└──────┬───────┘
       │
       │ Tests pass
       ▼
┌──────────────┐
│ Jenkins Job 2│
│              │
│    MERGE     │
│  dev → main  │
└──────┬───────┘
       │
       │ Merge successful
       ▼
┌──────────────┐
│ Jenkins Job 3│
│              │
│    DEPLOY    │
└──────┬───────┘
       │
       │ SSH + file transfer
       ▼
┌──────────────┐
│   AWS EC2    │
│              │
│ Run updated  │
│ application  │
└──────────────┘
```
## 10. Testing and Results

The pipeline was tested multiple times to confirm that changes made to the development branch could successfully progress through the CI/CD process and appear on the deployed application.

### Testing Process

The application front page was modified on the development branch and the changes were pushed to GitHub.

The GitHub webhook triggered Jenkins automatically, starting the CI/CD process.

The change then progressed through the three Jenkins jobs:

```text
Developer change
      ↓
GitHub dev branch
      ↓
Webhook
      ↓
Job 1 — Test
      ↓
Job 2 — Merge
      ↓
dev → main
      ↓
Job 3 — Deploy
      ↓
AWS EC2
      ↓
Updated front page

## 12. Benefits of the CI/CD Pipeline

### Benefits Observed

The CI/CD pipeline provided several benefits during the project:

- **Automation:** Testing, merging and deployment were performed automatically.

- **Faster feedback:** Developers could identify failed tests quickly.

- **Consistency:** The same deployment process was followed each time.

- **Reduced manual work:** Developers did not need to manually transfer and deploy every change.

- **Improved reliability:** Multiple pipeline runs could be used to verify that changes were deployed consistently.

- **Controlled deployments:** Code had to pass the testing stage before reaching the deployment stage.

### Benefits for an Organisation

A CI/CD pipeline can provide significant benefits to an organisation.

Automating repetitive processes can reduce the amount of manual work required from development and operations teams. This allows teams to spend more time working on application improvements rather than manually testing and deploying changes.

Automated testing also provides an early indication when a change has introduced a problem. This can reduce the risk of faulty code reaching users.

A repeatable deployment process can also improve consistency between deployments and reduce the possibility of human error.

Overall, CI/CD supports faster and more reliable software delivery while creating a controlled process for moving changes from development into a deployed environment.