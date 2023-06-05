# Workshop Exercise - Check the Prerequisites

**Read this in other languages**:
<br>![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png)[日本語](README.ja.md), ![brazil](../../../images/brazil.png) [Portugues do Brasil](README.pt-br.md), ![france](../../../images/fr.png) [Française](README.fr.md),![Español](../../../images/col.png) [Español](README.es.md).

## Table of Contents

- [Workshop Exercise - Check the Prerequisites](#workshop-exercise---check-the-prerequisites)
  - [Table of Contents](#table-of-contents)
  - [Objective](#objective)
  - [Introduction](#introduction)
    - [Understanding Execution Environments](#understanding-execution-environments)
    - [Understanding GitOps](#understanding-gitops)
  - [Guide](#guide)
    - [Your Lab Environment](#your-lab-environment)
    - [Step 1 - Access the Environment](#step-1---access-the-environment)
    - [Step 2 - Using the Terminal](#step-2---using-the-terminal)
    - [Step 3 - Examining Execution Environments](#step-3---examining-execution-environments)
    - [Step 4 - Examining the ansible-navigator configuration](#step-4---examining-the-ansible-navigator-configuration)
    - [Step 5 - Challenge Labs](#step-5---challenge-labs)

## Objective

This workshop aims to provide participants with an understanding of how EEs can be customized, created, and pushed to a central image registry using Openshift Pipelines. The following topics will be covered:

- Demonstrating the steps involved in customizing EEs using a Git repository.
- Demonstrating the steps involved in creating EEs using Openshift Pipelines.
- Demonstrating the steps involved in scanning and pushing EEs to an image registry (Private Ansible Automation Hub).
- Demonstrating the steps involved in using custom EEs pushed to the registry to run job templates from the Ansible Automation Controller.

## Introduction

The use of containers in automation is driven by the principles of reproducibility and consistency. Red Hat Ansible Automation Platform (AAP) also incorporates these principles through the use of YAML-formatted playbooks, allowing for uniform setup and configuration of thousands of instances.

By utilizing containers and maintaining a single source of truth, organizations can utilize the GitOps operational framework to apply DevOps best practices, such as version control, collaboration, compliance, and CI/CD tooling, to infrastructure automation.

In this workshop, we will explore the process of customizing, creating, and pushing EEs to a central image registry using OpenShift Pipelines. Participants will learn how to use a Git repository to customize EEs, create EEs using OpenShift Pipelines, scan and push EEs to a central image registry, and run job templates from the Automation Controller using custom EEs.

### Understanding Execution Environments

Automation Execution Environments (EEs) are container images on which all automation in Red Hat Ansible Automation Platform is run. They provide a defined, consistent, and portable environment for executing automation, and simplify the administration of Ansible Automation Platform for the platform administrator.

The best way to build EEs is by relying on a new tool provided by Red Hat® Ansible® Automation Platform, the `ansible-builder` command.

### Understanding GitOps

Infrastructure automation is essential to meet the demands of modern infrastructure. GitOps is a method used to automate the process of provisioning infrastructure, in this case, custom EEs. Similar to how developers use application source code, operations teams that adopt GitOps use configuration files stored as code (IaC - Infrastructure as Code) to generate a consistent infrastructure environment every time it is deployed.

The GitOps workflow consists of four key components:

- Git repository
- CI/CD pipeline
- A deployment tool
- A monitoring system

The Git repository serves as the source of truth for the configuration and code, the CI/CD pipeline is responsible for building, testing, and deploying the application, the deployment tool manages the application resources in the target environment, and the monitoring system tracks the status and performance of the deployed application.

![Ansible GitOps](ansible-gitops.png "Ansible GitOps")

## Guide

### Your Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                 | Inventory name |
| ---------------------| ---------------|
| Ansible Control Host | ansible-1      |
| Managed Host 1       | node1          |
| Managed Host 2       | node2          |
| Managed Host 3       | node3          |

### Step 1 - Access the Environment

<table>
<thead>
  <tr>
    <th>It is highly encouraged to use Visual Studio Code to complete the workshop exercises. Visual Studio Code provides:
    <ul>
    <li>A file browser</li>
    <li>A text editor with syntax highlighting</li>
    <li>A in-browser terminal</li>
    </ul>
    Direct SSH access is available as a backup, or if Visual Studio Code is not sufficient to the student.  There is a short YouTube video provided if you need additional clarity: <a href="https://youtu.be/Y_Gx4ZBfcuk">Ansible Workshops - Accessing your workbench environment</a>.
</th>
</tr>
</thead>
</table>

- Connect to Visual Studio Code from the Workshop launch page (provided by your instructor).  The password is provided below the WebUI link.

  ![launch page](images/launch_page.png)

- Type in the provided password to connect.

  ![login vs code](images/vscode_login.png)

  - Open the `rhel-workshop` directory in Visual Studio Code:

### Step 2 - Using the Terminal

- Open a terminal in Visual Studio Code:

  ![picture of new terminal](images/vscode-new-terminal.png)

Navigate to the `rhel-workshop` directory on the Ansible control node terminal.

```bash
[student@ansible-1 ~]$ cd ~/rhel-workshop/
[student@ansible-1 rhel-workshop]$ pwd
/home/student/rhel-workshop
[student@ansible-1 rhel-workshop]$
```

* `~` - the tilde in this context is a shortcut for the home directory, i.e. `/home/student`
* `cd` - Linux command to change directory
* `pwd` - Linux command for print working directory.  This will show the full path to the current working directory.

### Step 3 - Examining Execution Environments

Run the `ansible-navigator` command with the `images` argument to look at execution environments configured on the control node:

```bash
$ ansible-navigator images
```

![ansible-navigator images](images/navigator-images.png)


> Note: The output  you see might differ from the above output

This command gives you information about all currently installed Execution Environments or EEs for short.  Investigate an EE by pressing the corresponding number.  For example pressing **2** with the above example will open the `ee-supported-rhel8` execution environment:

![ee main menu](images/navigator-ee-menu.png)

Selecting `2` for `Ansible version and collections` will show us all Ansible Collections installed on that particular EE, and the version of `ansible-core`:

![ee info](images/navigator-ee-collections.png)

### Step 4 - Examining the ansible-navigator configuration

Either use Visual Studio Code to open or use the `cat` command to view the contents of the `ansible-navigator.yml` file.  The file is located in the home directory:

```bash
$ cat ~/.ansible-navigator.yml
---
ansible-navigator:
  ansible:
    inventory:
      entries:
      - /home/student/lab_inventory/hosts

  execution-environment:
    image: registry.redhat.io/ansible-automation-platform-20-early-access/ee-supported-rhel8:2.0.0
    enabled: true
    container-engine: podman
    pull:
      policy: missing
    volume-mounts:
    - src: "/etc/ansible/"
      dest: "/etc/ansible/"
```

Note the following parameters within the `ansible-navigator.yml` file:

* `inventories`: shows the location of the ansible inventory being used
* `execution-environment`: where the default execution environment is set

For a full listing of every configurable knob checkout the [documentation](https://ansible-navigator.readthedocs.io/en/latest/settings/)

### Step 5 - Challenge Labs

You will soon discover that many chapters in this lab guide come with a "Challenge Lab" section. These labs are meant to give you a small task to solve using what you have learned so far. The solution of the task is shown underneath a warning sign.

---
**Navigation**

<br>

{% if page.url contains 'ansible_rhel_90' %}
[Next Exercise](../2-thebasics)
{% else %}
[Next Exercise](../1.2-thebasics)
{% endif %}
<br><br>
[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md)
