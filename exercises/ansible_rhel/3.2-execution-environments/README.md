# Workshop Exercise - Exploring Execution Environments

**Read this in other languages**:
<br>![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png)[日本語](README.ja.md), ![brazil](../../../images/brazil.png) [Portugues do Brasil](README.pt-br.md), ![france](../../../images/fr.png) [Française](README.fr.md),![Español](../../../images/col.png) [Español](README.es.md).

## Table of Contents

- [Workshop Exercise - Exploring Execution Environments](#workshop-exercise---exploring-execution-environments)
  - [Table of Contents](#table-of-contents)
  - [Objective](#objective)
  - [Guide](#guide)
    - [Step 1 - Use Automation Execution Environments](#step-1---use-automation-execution-environments)
      - [Inspect Automation Execution Environments](#inspect-automation-execution-environments)
      - [Using Automation Execution Environments](#using-automation-execution-environments)
    - [Step 2 - Build Automation Execution Environments](#step-2---build-automation-execution-environments)
      - [Deciding When to Create a Custom Automation Execution Environment](#deciding-when-to-create-a-custom-automation-execution-environment)
      - [Preparing for a New Automation Execution Environment](#preparing-for-a-new-automation-execution-environment)
        - [Ansible Content Collections to Install with `requirements.yml`](#ansible-content-collections-to-install-with-requirementsyml)
        - [Python Packages to Install with `requirements.txt`](#python-packages-to-install-with-requirementstxt)
        - [RPM Packages to Install with `bindep.txt`](#rpm-packages-to-install-with-bindeptxt)
      - [Building a New Automation Execution Environment](#building-a-new-automation-execution-environment)
    - [Takeaways](#takeaways)

## Objective

This exercise will help users understand how to identify the Automation Execution
Environments and select the correct one for your use case.

An *automation execution environment* is a container image that includes Ansible Content Collections, their software dependencies, and a minimal Ansible engine that can run your playbooks.
By using an automation execution environment, you can use the same consistent, portable environment to develop your Ansible Playbooks on one system and run them on another. This streamlines and simplifies the development process and helps to ensure predictable, reproducible results.

The automation execution environment is where your Ansible Playbook actually runs. You normally use a tool such as `ansible-navigator` to run a playbook, but the playbook runs inside the container rather than directly on your system.

An automation execution environment consists of the following:

- Ansible Core (or Ansible).
- Ansible Content Collections to supplement Ansible Core.
- Python and any other dependencies of Ansible Core and the included collections.
- Ansible Runner to run your playbooks.

## Guide

### Step 1 - Use Automation Execution Environments

#### Inspect Automation Execution Environments

Use the automation content navigator to list and inspect the automation execution environments available on the system. To do so, run the `ansible-navigator images` command:

```bash
ansible-navigator images
```

From here, we can inspect the execution environment. For example, pressing 2 will show the details of the execution environment.

#### Using Automation Execution Environments

The `ansible-navigator run` command runs your playbooks in an automation execution environment.

That tool uses the supported automation execution environment by default, but you can select a different one by specifying the `--execution-environment-image` (or `--eei`) option. The following example shows how to run a playbook using the compatibility automation execution environment:

```bash
ansible-navigator run oldplaybook.yml --eei registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8:1.0
```

If the container image is already available on your system, then you can use its short image name: `--eei ee-29-rhel8:1.0` in the preceding example.

```bash
ansible-navigator run oldplaybook.yml --eei ansible-automation-platform/ee29-rhel8:2.9
```

If the container image is not already available on your system, `ansible-navigator` tries to pull it from the container registry. You need to make sure that you are authenticated to that registry with the `podman login` command first. In the preceding example, at the beginning of your session, run the podman login registry.redhat.io command and provide your Customer Portal credentials to authenticate to the registry.

> **NOTE**
>
> Instead of using the `--eei` option, you can create an `ansible-navigator.yml` configuration file to define the automation execution environment to use by default.

```yaml
---
- name: Retrieving ACLs using the ansible.posix collection
  hosts: web_servers
  gather_facts: false
  tasks:
    - name: Obtain the ACLs for /webserver_data directory
      ansible.posix.acl:
        path: /webserver_data/
        register: acl_info
    - name: Print the ACLs obtained for /webserver_data directory
      ansible.builtin.debug:
        var: acl_info.acl
```

1. Identify the modules that the playbook uses.

  - ansible.posix.acl
  - ansible.builtin.debug

To successfully run the prebuilt playbook, you need the `ansible.posix.acl` module, which is part of the `ansible.posix` Ansible Content Collection.

2. Find the automation execution environment that already includes the `ansible.posix` Ansible Content Collection.

> **NOTE**
>
> Use the `podman login` command to log in to the private automation hub at `registry.redhat.io` if you haven't done yet.
>
> `podman login registry.redhat.io`

3. Use the `ansible-navigator images` command to pull the minimal automation execution environment and analyze it.

```shell
ansible-navigator images --eei ansible-automation-platform/ee-minimal-rhel8:2.14
```

4. Press 0 to inspect this automation execution environment. Then, press 2 to select the **Ansible version and collections** option.

5. The resulting screen indicates that the minimal automation execution environment **does not** contain any additional Ansible Content Collections.

6. Type `:q` and press `Enter` to exit the `ansible-navigator` command. The `ee-minimal-rhel8` automation execution environment **is not useful for running the prebuilt playbook**.

7. Use the `ansible-navigator` command to pull down and inspect the supported automation execution environment.

```shell
ansible-navigator images --eei ansible-automation-platform-23 /ee-supported-rhel8:1.0
```

8. Press 1 to inspect this automation execution environment. Then, press 2 to select the **Ansible version and collections** option.

9. The resulting screen indicates that the supported automation execution environment contains the `ansible.posix` Ansible Content Collection.

10. Type `:q` and press `Enter` to exit the `ansible-navigator` command. The `ee-supported-rhel8` automation execution environment contains the `ansible.posix` Ansible Content Collection that you need. **You can run the prebuilt playbook using this automation execution environment.**

11. As a demonstration that the *minimal* automation execution environment does not work with the `acl_info.yml` playbook because it does not include the `ansible.posix` collection. Use the `ansible-navigator` command to run the `acl_info.yml` playbook using the minimal automation execution environment.

```shell
ansible-navigator run acl_info.yml > -m stdout --eei ansible-automation-platform/ee-minimal-rhel8:2.14
```

As expected, the playbook cannot resolve the ansible.posix.acl module, causing it to fail.

12. Use the `ansible-navigator` command to run the `acl_info.yml` playbook using the `ee-supported-rhel8:1.0` automation execution environment.

```shell
ansible-navigator run acl_info.yml -m stdout --eei ansible-automation-platform/ee-supported-rhel8:1.0
```

This automation execution environment contains the ansible.posix collection and the command succeeds.

### Step 2 - Build Automation Execution Environments

#### Deciding When to Create a Custom Automation Execution Environment

Red Hat provides several Ansible automation execution environments that suit the needs of many users. These automation execution environments include the most common Ansible Content Collections.

You can use the `ansible-navigator images` command to inspect a container image and view the collections, Python packages, and operating system packages it includes.

The automation execution environments that Red Hat provides might be sufficient for your playbooks. In that case, you do not need to create new automation execution environments.

Sometimes, you might need an Ansible Content Collection that is not included in one of the existing automation execution environments. Often, you can install these extra collections in your Ansible project and then use an existing automation execution environment without having to build a new one.

On the other hand, you should consider creating a custom automation execution environment in the following situations:

- You frequently reuse specific collections that are not included in the existing automation execution environments. It is more efficient to embed those collections in a custom automation execution environment than to install them from the project `requirements.yml` file every time you run a playbook in that project. This is especially true if you have many projects that use that collection.
- The collection you want to use requires Python packages or software that are not included in an existing automation execution environment.
- You need to use a collection that conflicts with another collection in your existing automation execution environments.

#### Preparing for a New Automation Execution Environment

You use the `ansible-builder` command to create the container images used for automation execution environments.
The ansible-builder RPM package provides that command.

```shell
sudo yum install ansible-builder
```

Create a working directory to prepare the files that you need to build the automation execution environment container image. The `ansible-builder` command searches this working directory for its `execution-environment.yml` configuration file, which it uses to determine how to build the container image.

```shell
mkdir my-execution-environment
cd my-execution-environment
```

```yaml
---
version: 1
build_arg_defaults:
  EE_BASE_IMAGE: registry.redhat.io/ansible-automation-platform/ee-minimal-rhel8:2.14
  EE_BUILDER_IMAGE: registry.redhat.io/ansible-automation-platform/ansible-builder-rhel8:1.2
ansible_config: ansible.cfg
dependencies:
  galaxy: requirements.yml
  python: requirements.txt
  system: bindep.txt
```

- The **EE_BASE_IMAGE** parameter specifies the automation execution environment container image to use as the starting point.
- The **EE_BUILDER_IMAGE** parameter points to a container image that includes the tools that the build process uses to construct automation execution environment container images. You can omit this parameter.
- The optional `ansible_config` parameter specifies an Ansible configuration file. The build process uses that file to pull collections from a location that requires authentication.
- The **galaxy** parameter points to the file that lists the collections to install inside the automation execution environment. Ansible users usually set the file name to `requirements.yml`.
- The **python** parameter points to the file that lists the Python packages to install. Some collections include a `requirements.txt` file. If you install such a collection, then you do not need to list its Python dependencies. Python developers usually set the file name to `requirements.txt`.
- The **system** parameter points to the file that lists the RPM packages to install. Some collections include a `bindep.txt` file. If you install such a collection, then you do not need to list its RPM dependencies. Ansible users usually set the file name to `bindep.txt`, which is also the name used by the bindep tool that processes the file.

##### Ansible Content Collections to Install with `requirements.yml`

In the `requirements.yml` file, list the content collections that ansible-builder should install inside the automation execution environment.

The following example is a straightforward `requirements.yml` file that instructs the `ansible-builder` command to install the `community.aws` and `community.general` collections from Ansible Galaxy.

```yaml
collections:
  - community.aws
  - community.general
```

##### Python Packages to Install with `requirements.txt`

In the `requirements.txt` file, list the Python packages that `ansible-builder` must install inside the automation execution environment.

The following example `requirements.txt` file lists some required Python packages. For each package, you can provide a specific version to select.

```ini
sh==1.13.1
jsonschema>=3.2.0,<4.0.1
textfsm
ttp
xmltodict
dnspython
```

##### RPM Packages to Install with `bindep.txt`

In the `bindep.txt` file, list the RPM packages `ansible-builder` must install inside the automation execution environment.
The following `bindep.txt` file lists the RPM packages required by the preceding examples.

```ini
rsync [platform:rpm]
kubernetes-client [platform:rpm]
```

> **NOTE**
>
> The bindep tool, which processes the `bindep.txt` file, can manage packages for several Linux distributions. Different distributions might have other package names for the same software. In the preceding file, the `[platform:rpm]` directive targets all the Linux distributions that use the RPM packaging system.

#### Building a New Automation Execution Environment

The execution environment builder automatically pulls the base and the builder images if they are not already available locally.
If you use images from a container registry that requires authentication, then you must authenticate before starting the build process.

> **NOTE**
>
> Use the `podman login` command to log in to the private automation hub at `registry.redhat.io` if you haven't done yet.
>
> `podman login registry.redhat.io`

After you have prepared your configuration files and authenticated to the container registry, run the `ansible-builder build` command to create your automation execution environment container image.
Use the `--tag` (or `-t`) option to provide a name for the container image. For example, use the `--tag demo:v1.0` option to name the container **demo** and give it the *v1.0* tag.
A successful build creates a new container image.

```shell
cd my-execution-environment
cat execution-environment.yml
cat requirements.yml
cat requirements.txt
cat bindep.txt
ansible-builder build --tag my-ee-demo:v1.0
```

Use the podman images command to display local container images and discover your new created image.

```shell
podman images
```

### Takeaways

- You can use the ansible-navigator images command to inspect automation execution environments and to list the collections and their dependencies.
- Ansible Playbooks refer to modules, roles, and plug-ins in Ansible Content Collections by their fully qualified collection name (FQCN).
- Automation execution environments can access the Ansible Content Collections on the local system that are installed in a collections/ directory in the same directory as the playbook.
- The supported automation execution environment is used by default by automation content navigator and automation controller.
- The minimal automation execution environment only provides the ansible.builtin Ansible Content Collection, but you can use others from your collections/ directory.
- You can use the compatibility automation execution environment for playbooks that require an older version of Ansible.

---
**Navigation**
<br>
[Previous Exercise](../3.1-collections) - [Next Exercise](../3.3-gitops-automation/)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
