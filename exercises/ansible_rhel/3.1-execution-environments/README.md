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
[student@ansible-1 ~]$ ansible-navigator images

  Image                                                                           Tag                                       Execution environment                                                                        Created                                               Size
0│ee-supported-rhel8                                                              1.0.0-208                                 True                                                                                         6 months ago                                          1.66 GB



^b/PgUp page up                                       ^f/PgDn page down                                       ↑↓ scroll                                       esc back                                       [0-9] goto                                       :help help
```

From here, we can inspect the execution environment.
To select the existing Execution Environment, we need to press `**0** ee-supported-rhel8`, and then pressing `**0** Image information` will show the details of the execution environment.

If we need to check the collections or ansible information from the image, we can press `**2** Ansible version and collections`


```bash
 0│---
 1│ansible:
 2│  collections:
 3│    details:
 4│      amazon.aws: 5.1.0
 5│      ansible.controller: 4.3.0
 6│      ansible.netcommon: 4.1.0
 7│      ansible.network: 1.2.0
 8│      ansible.posix: 1.3.0
 9│      ansible.security: 1.0.0
10│      ansible.snmp: 1.0.1
11│      ansible.utils: 2.7.0
12│      ansible.windows: 1.12.0
13│      ansible.yang: 1.0.0
14│      arista.eos: 6.0.0
15│      cisco.asa: 4.0.0
16│      cisco.ios: 4.1.0
17│      cisco.iosxr: 4.0.2
18│      cisco.nxos: 4.0.0
19│      cloud.common: 2.1.2
20│      frr.frr: 2.0.0
21│      ibm.qradar: 2.1.0
22│      junipernetworks.junos: 4.0.0
23│      kubernetes.core: 2.3.2
24│      openvswitch.openvswitch: 2.1.0
25│      redhat.insights: 1.0.7
26│      redhat.openshift: 2.2.0
27│      redhat.rhv: 2.1.0
28│      redhat.satellite: 3.3.0
29│      servicenow.itsm: 1.3.3
30│      splunk.es: 2.1.0
31│      trendmicro.deepsec: 2.0.0
32│      vmware.vmware_rest: 2.2.0
33│      vyos.vyos: 4.0.0
34│  version:
35│    details: ansible [core 2.14.1]
```

> **NOTE**
>
> That the Execution Environment Image `ee-supported-rhel8` is providing us `ansible-core 2.14.1` and a minimal set of collections.
>

To configure the default Execution Environment Image you can edit the file `~/.ansible-navigator.yml` under the `execution-environment/image` section, for this specific environment looks like:

```yaml
---
ansible-navigator:
  ansible:
    inventory:
      entries:
      - /home/student/lab_inventory/hosts

  execution-environment:
    image: registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8:1.0.0-208
    enabled: true
    container-engine: podman
    pull:
      policy: missing
    volume-mounts:
    - src: "/etc/ansible/"
      dest: "/etc/ansible/"
```

#### Using Automation Execution Environments

The `ansible-navigator run` command runs your playbooks in an automation execution environment, the default one is the one defined on the file `~/.ansible-navigator.yml`, but you can select a different one by specifying the `--execution-environment-image` (or `--eei`) option. The q following example shows how to run a playbook using the compatibility automation execution environment:

```bash
[student@ansible-1 ~]$ echo "---
- name: Update web servers
  hosts: web
  become: true

  tasks:
  - name: Ensure apache is at the latest version
    yum:
      name: httpd
      state: latest
  - name: Ensure the apache service is started and enabled
    systemd:
      name: httpd
      state: started
      enabled: true" old-playbook.yml
```

> **NOTE**
>
> The tasks on the `old-playbook.yml` file are **NOT** using the FQCN (Full Qualify Collection Name) for the modules, that is the *old way* to write ansible playbooks, as the new method includes the FQCN for each module, for this example, the `yum` module should become `ansible.builtin.yum` and `systemd` becomes `ansible.builtin.systemd`
>

The next command is going to run the `old-playbook.yml` using an Execution Environment that uses an older version of ansible, in this case `Ansible 2.9`

```bash
[student@ansible-1 ~]$ ansible-navigator run oldplaybook.yml --eei registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8:1.0
----------------------------------------------------------------------------------------------------
Execution environment image and pull policy overview
----------------------------------------------------------------------------------------------------
Execution environment image name:     registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8:1.0
Execution environment image tag:      1.0
Execution environment pull arguments: None
Execution environment pull policy:    missing
Execution environment pull needed:    True
----------------------------------------------------------------------------------------------------
Updating the execution environment
----------------------------------------------------------------------------------------------------
Running the command: podman pull registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8:1.0
Trying to pull registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8:1.0...
Getting image source signatures
Checking if image destination supports signatures
Copying blob bcccb9fc293e done
Copying blob 2ad3b9180fba done
Copying blob 78159f5d6288 done
Copying blob 28ff5ee6facb done
Copying config 58c796a146 done
Writing manifest to image destination
Storing signatures
58c796a146e35be2d61a175abadeb5ad9a5f76189b7d9e4285ad1d56a52e3731

PLAY [Update web servers] ******************************************************

TASK [Gathering Facts] *********************************************************
ok: [node1]
ok: [node2]
ok: [node3]

TASK [Ensure apache is at the latest version] **********************************
changed: [node1]
changed: [node2]
changed: [node3]

TASK [Ensure the apachhe service is started and enabled] ***********************
changed: [node1]
changed: [node2]
changed: [node3]

PLAY RECAP *********************************************************************
node1                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node2                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node3                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

> **NOTE**
>
> As we don't have the Execution Environment Image `registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8:1.0` in our system, the previous command downloaded it from the Registry `registry.redhat.io` and now we can refer to that Execution Environment Images as `ansible-automation-platform-23/ee-29-rhel8:1.0` or `ee-29-rhel8`
> Example:
> ```bash
>[student@ansible-1 ~]$ ansible-navigator run oldplaybook.yml --eei ee29-rhel8
>```

> **NOTE**
>
> If the Execution Environment Image is not already available on your system, `ansible-navigator` tries to pull it from the container registry. You need to make sure that you are authenticated to that registry with the `podman login` command first. In the preceding example, at the beginning of your session, run the `podman login registry.redhat.io` command and provide your Customer Portal credentials to authenticate to the registry.
>

To confirm that indeed we are using `ansible 2.9.27` we can run:

```bash
[student@ansible-1 ~]$ ansible-navigator images
Image                                                                           Tag                                       Execution environment                                                                        Created                                               Size
0│ee-29-rhel8                                                                     1.0                                       True                                                                                         12 days ago                                           888 MB
1│ee-supported-rhel8                                                              1.0.0-208                                 True                                                                                         6 months ago                                          1.66 GB
```

Then press `**0** ee-29-rhel8` and then `**2** Ansible version and collections`

```
 0│---
 1│ansible:
 2│  collections:
 3│    details: This command is not supported with ansible 2.9.
 4│    errors:
 5│    - |-
 6│      usage: ansible-galaxy collection [-h] COLLECTION_ACTION ...
 7│      ansible-galaxy collection: error: argument COLLECTION_ACTION: invalid choice: 'list' (choose from 'init', 'build', 'publish', 'install')
 8│  version:
 9│    details: ansible 2.9.27
```

> **NOTE**
>
> That the Execution Environment Image `ee-29-rhel8` is providing us `ansible 2.9.27` and **NO** collections at all.
>

> **NOTE**
>
> Instead of using the `--eei` option, you can create an `ansible-navigator.yml` configuration file to define the automation execution environment to use by default.


With the previous steps in mind, lets review the following playbook

```yaml
---
- name: Retrieving ACLs using the ansible.posix collection
  hosts: web
  gather_facts: false
  tasks:
    - name: Obtain the ACLs for /var/www/html directory
      ansible.posix.acl:
        path: /var/www/html
        register: acl_info
    - name: Print the ACLs obtained for /var/www/html directory
      ansible.builtin.debug:
        var: acl_info.acl
```

1. Identify the modules that the playbook uses.

- ansible.posix.acl
- ansible.builtin.debug

To successfully run the prebuilt playbook, you need the `ansible.posix.acl` module, which is part of the `ansible.posix` Ansible Content Collection in the Execution Environment Image.

2. Find the automation execution environment that already includes the `ansible.posix` Ansible Content Collection.

> **NOTE**
>
> Use the `podman login` command to log in to the private automation hub at `registry.redhat.io` if you haven't done yet.
>
> `podman login registry.redhat.io`

3. Use the `ansible-navigator images` command to pull the minimal automation execution environment and analyze it.

```shell
[student@ansible-1 ~]$ ansible-navigator images --eei ansible-automation-platform/ee-minimal-rhel8:2.14
------------------------------------------------------------------------------------
Execution environment image and pull policy overview
------------------------------------------------------------------------------------
Execution environment image name:     ansible-automation-platform/ee-minimal-rhel8:2.14
Execution environment image tag:      2.14
Execution environment pull arguments: None
Execution environment pull policy:    missing
Execution environment pull needed:    True
------------------------------------------------------------------------------------
Updating the execution environment
------------------------------------------------------------------------------------
Running the command: podman pull ansible-automation-platform/ee-minimal-rhel8:2.14
Resolved "ansible-automation-platform/ee-minimal-rhel8" as an alias (/etc/containers/registries.conf.d/001-rhel-shortnames.conf)
Trying to pull registry.redhat.io/ansible-automation-platform/ee-minimal-rhel8:2.14...
Getting image source signatures
Checking if image destination supports signatures
Copying blob b3cd148e9e0a done
Copying blob d2b5f358ecf1 done
Copying blob 4104854ab26c done
Copying blob f6ce0e898c3a done
Copying config ab66adc818 done
Writing manifest to image destination
Storing signatures
ab66adc818632fb4046f579ba8c65a0a186b51be88e5335dea1ae7a8aa11d797

  Image                                                                           Tag                                       Execution environment                                                                        Created                                               Size
0│ee-29-rhel8                                                                     1.0                                       True                                                                                         12 days ago                                           888 MB
1│ee-minimal-rhel8                                                                2.14                                      True                                                                                         3 weeks ago                                           306 MB
2│ee-supported-rhel8                                                              1.0.0-208                                 True                                                                                         6 months ago                                          1.66 GB



^b/PgUp page up                                       ^f/PgDn page down                                       ↑↓ scroll                                       esc back                                       [0-9] goto                                       :help help
```

4. Press `**1**` to inspect this automation Execution Environment Image. Then, press `**2**` to select the **Ansible version and collections** option.

```
0│---
1│ansible:
2│  collections:
3│    details: {}
4│    errors:
5│    - |-
6│      ERROR! - None of the provided paths were usable. Please specify a valid path with --collections-path
7│  version:
8│    details: ansible [core 2.14.6]
```

5. The resulting screen indicates that the minimal automation execution environment **does not** contain any additional Ansible Content Collections.

6. Type `:q` and press `Enter` to exit the `ansible-navigator` command. The `ee-minimal-rhel8` automation execution environment **is not useful for running the prebuilt playbook**.

7. Use the `ansible-navigator` command to pull down and inspect the supported automation execution environment.

```shell
[student@ansible-1 ~]$ ansible-navigator images --eei registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8:1.0
-----------------------------------------------------------------------------------------------------------
Execution environment image and pull policy overview
-----------------------------------------------------------------------------------------------------------
Execution environment image name:     registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8:1.0
Execution environment image tag:      1.0
Execution environment pull arguments: None
Execution environment pull policy:    missing
Execution environment pull needed:    True
-----------------------------------------------------------------------------------------------------------
Updating the execution environment
-----------------------------------------------------------------------------------------------------------
Running the command: podman pull registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8:1.0
Trying to pull registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8:1.0...
Getting image source signatures
Checking if image destination supports signatures
Copying blob d2b5f358ecf1 skipped: already exists
Copying blob debf4e5df458 done
Copying blob 749eb51af9d8 done
Copying blob 4029f3faeb45 done
Copying config 36a5c93b87 done
Writing manifest to image destination
Storing signatures
36a5c93b87ddb436b21eccc41a55c81b79e550525754c1af2d2323bd5a4b21b3

 Image                                                                           Tag                                       Execution environment                                                                        Created                                               Size
0│ee-29-rhel8                                                                     1.0                                       True                                                                                         12 days ago                                           888 MB
1│ee-minimal-rhel8                                                                2.14                                      True                                                                                         3 weeks ago                                           306 MB
2│ee-supported-rhel8                                                              1.0                                       True                                                                                         12 days ago                                           1.68 GB
3│ee-supported-rhel8                                                              1.0.0-208                                 True                                                                                         6 months ago                                          1.66 GB



^b/PgUp page up                                       ^f/PgDn page down                                       ↑↓ scroll                                       esc back                                       [0-9] goto                                       :help help
```

8. Press `**2**` to inspect this automation Execution Environment Image. Then, press `**2**` to select the **Ansible version and collections** option.

```
 0│---
 1│ansible:
 2│  collections:
 3│    details:
 4│      amazon.aws: 5.1.0
 5│      ansible.controller: 4.3.0
 6│      ansible.netcommon: 4.1.0
 7│      ansible.network: 1.2.0
 8│      ansible.posix: 1.3.0
 9│      ansible.security: 1.0.0
10│      ansible.snmp: 1.0.1
11│      ansible.utils: 2.7.0
12│      ansible.windows: 1.12.0
13│      ansible.yang: 1.0.0
14│      arista.eos: 6.0.0
15│      cisco.asa: 4.0.0
16│      cisco.ios: 4.4.0
17│      cisco.iosxr: 5.0.0
18│      cisco.nxos: 4.1.0
19│      cloud.common: 2.1.2
20│      frr.frr: 2.0.0
21│      ibm.qradar: 2.1.0
22│      junipernetworks.junos: 4.0.0
23│      kubernetes.core: 2.3.2
24│      openvswitch.openvswitch: 2.1.0
25│      redhat.insights: 1.0.7
26│      redhat.openshift: 2.2.0
27│      redhat.rhv: 2.1.0
28│      redhat.satellite: 3.3.0
29│      servicenow.itsm: 1.3.3
30│      splunk.es: 2.1.0
31│      trendmicro.deepsec: 2.0.0
32│      vmware.vmware_rest: 2.2.0
33│      vyos.vyos: 4.0.0
34│  version:
35│    details: ansible [core 2.14.6]
```

9.  The resulting screen indicates that the supported automation execution environment contains the `ansible.posix` Ansible Content Collection.

10. Type `:q` and press `Enter` to exit the `ansible-navigator` command. The `ee-supported-rhel8` automation execution environment contains the `ansible.posix` Ansible Content Collection that you need. **You can run the prebuilt playbook using this automation execution environment.**

11. As a demonstration that the *minimal* automation execution environment does not work with the `acl_info.yml` playbook because it does not include the `ansible.posix` collection. Use the `ansible-navigator` command to run the `acl_info.yml` playbook using the minimal automation execution environment.

```shell
ansible-navigator run acl_info.yml -m stdout --eei ee-minimal-rhel8:2.14
ERROR! couldn't resolve module/action 'ansible.posix.acl'. This often indicates a misspelling, missing collection, or incorrect module path.

The error appears to be in '/home/student/acl_info.yml': line 6, column 7, but may
be elsewhere in the file depending on the exact syntax problem.

The offending line appears to be:

  tasks:
    - name: Obtain the ACLs for /var/www/html directory
      ^ here
Please review the log for errors.
```

As expected, the playbook cannot resolve the ansible.posix.acl module, causing it to fail.

12. Use the `ansible-navigator` command to run the `acl_info.yml` playbook using the `ee-supported-rhel8:1.0` automation execution environment.

```shell
[student@ansible-1 ~]$ ansible-navigator run acl_info.yml -m stdout --eei ee-supported-rhel8:1.0

PLAY [Retrieving ACLs using the ansible.posix collection] **********************

TASK [Obtain the ACLs for /var/www/html directory] *****************************
ok: [node3]
ok: [node2]
ok: [node1]

TASK [Print the ACLs obtained for /var/www/html directory] *********************
ok: [node1] => {
    "acl_info.acl": [
        "user::rwx",
        "group::r-x",
        "other::r-x"
    ]
}
ok: [node2] => {
    "acl_info.acl": [
        "user::rwx",
        "group::r-x",
        "other::r-x"
    ]
}
ok: [node3] => {
    "acl_info.acl": [
        "user::rwx",
        "group::r-x",
        "other::r-x"
    ]
}

PLAY RECAP *********************************************************************
node1                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node2                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
node3                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

This automation execution environment contains the `ansible.posix` collection and the command succeeds.

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
[student@ansible-1 ~]$ sudo yum install ansible-builder
```

Create a working directory to prepare the files that you need to build the automation execution environment container image. The `ansible-builder` command searches this working directory for its `execution-environment.yml` configuration file, which it uses to determine how to build the container image.

```shell
mkdir ~/execution-environment
cd execution-environment
```

```shell
[student@ansible-1 ~]$ echo "---
version: 1
build_arg_defaults:
  EE_BASE_IMAGE: registry.redhat.io/ansible-automation-platform/ee-minimal-rhel8:2.14
  EE_BUILDER_IMAGE: registry.redhat.io/ansible-automation-platform/ansible-builder-rhel8:1.2
ansible_config: ansible.cfg
dependencies:
  galaxy: requirements.yml
  python: requirements.txt
  system: bindep.txt " > execution-environment.yml
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
---
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
curl [platform:rpm]
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
cd execution-environment
cat execution-environment.yml
cat requirements.yml
cat requirements.txt
cat bindep.txt
[student@ansible-1 ~]$ ansible-builder build --tag my-ee-demo:v1.0
```

Use the podman images command to display local container images and discover your new created image.

```shell
[student@ansible-1 ~]$ podman images
REPOSITORY                                                            TAG         IMAGE ID      CREATED             SIZE
localhost/my-ee-demo                                                  v1.0        5a63e5230497  28 seconds ago      452 MB
<none>                                                                <none>      d9d839ff7a32  About a minute ago  485 MB
<none>                                                                <none>      59cd06e0aa9f  2 minutes ago       339 MB
<none>                                                                <none>      cdf36a0f164b  20 hours ago        253 MB
<none>                                                                <none>      439c692fca7b  20 hours ago        339 MB
<none>                                                                <none>      433c8492ade2  20 hours ago        253 MB
<none>                                                                <none>      af80478bb0db  20 hours ago        339 MB
<none>                                                                <none>      79575353b239  20 hours ago        253 MB
<none>                                                                <none>      75ac926d2a3a  20 hours ago        339 MB
registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8  1.0         36a5c93b87dd  13 days ago         1.68 GB
registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8         1.0         58c796a146e3  13 days ago         888 MB
registry.redhat.io/ansible-automation-platform/ee-minimal-rhel8       2.14        ab66adc81863  3 weeks ago         306 MB
registry.redhat.io/ansible-automation-platform/ansible-builder-rhel8  1.2         fa3b605f379b  6 weeks ago         220 MB
registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8  1.0.0-208   a36109621782  6 months ago        1.66 GB
```

Also, we can verify it with the `ansible-navigator images` command:

```shell
[student@ansible-1 ~]$ ansible-navigator images
  Image                                                                       Tag                                Execution environment                                                       Created                                         Size
0│ansible-builder-rhel8                                                       1.2                                False                                                                       6 weeks ago                                     220 MB
1│ee-29-rhel8                                                                 1.0                                True                                                                        13 days ago                                     888 MB
2│ee-minimal-rhel8                                                            2.14                               True                                                                        3 weeks ago                                     306 MB
3│ee-supported-rhel8                                                          1.0                                True                                                                        13 days ago                                     1.68 GB
4│ee-supported-rhel8                                                          1.0.0-208                          True                                                                        6 months ago                                    1.66 GB
5│my-ee-demo                                                                  v1.0                               True                                                                        2 minutes ago                                   452 MB


^b/PgUp page up                                ^f/PgDn page down                                ↑↓ scroll                                esc back                                [0-9] goto                                :help help
```

Let's build an Execution Environment Image for the next section, where we will want to use some custom collections

Let's start

1. Create a new directory named `ee-collections`

```shell
[student@ansible-1 ~]$ mkdir ~/ee-collections
[student@ansible-1 ~]$ cd ee-collections
[student@ansible-1 ee-collections]$
```

2. Create the files:
   1. `execution-environment.yml`
   2. `ansible.cfg`
   3. `requirements.yml`
   4. `requirements.txt`
   5. `bindep.txt`

```shell
[student@ansible-1 ee-collections]$ echo "---
version: 1
build_arg_defaults:
  EE_BASE_IMAGE: registry.redhat.io/ansible-automation-platform/ee-minimal-rhel8:2.14
  EE_BUILDER_IMAGE: registry.redhat.io/ansible-automation-platform/ansible-builder-rhel8:1.2
ansible_config: ansible.cfg
dependencies:
  galaxy: requirements.yml
  python: requirements.txt
  system: bindep.txt " > execution-environment.yml
```

```shell
[student@ansible-1 ee-collections]$ echo "
[defaults]
stdout_callback = yaml
connection = smart
timeout = 60
deprecation_warnings = False
action_warnings = False
system_warnings = False
devel_warning = False
host_key_checking = False
collections_on_ansible_version_mismatch = ignore
retry_files_enabled = False
interpreter_python = auto_silent
[persistent_connection]
connect_timeout = 200
command_timeout = 200
" > ansible.cfg
```

```shell
[student@ansible-1 ee-collections]$ echo "---
collections:
  - newswangerd.collection_demo
  - community.crypto
  - ansible.netcommon
  - ansible.posix
" > requirements.yml
```

```shell
[student@ansible-1 ee-collections]$ echo "---
awxkit
boto3
kubernetes
PyYAML
" > requirements.txt
```

```shell
[student@ansible-1 ee-collections]$ echo "
rsync [platform:rpm]
curl [platform:rpm]
" > bindep.txt
```

```shell
[student@ansible-1 ee-collections]$ ls -la
total 24
drwxrwxr-x.  2 student student  124 Jun 21 18:18 .
drwx------. 14 student student 4096 Jun 21 17:30 ..
-rw-rw-r--.  1 student student  371 Jun 21 18:17 ansible.cfg
-rw-rw-r--.  1 student student   43 Jun 21 18:17 bindep.txt
-rw-rw-r--.  1 student student  332 Jun 21 18:18 execution-environment.yml
-rw-rw-r--.  1 student student   56 Jun 21 18:17 requirements.txt
-rw-rw-r--.  1 student student  152 Jun 21 18:17 requirements.yml
```

Once all the files are in place, lets build a new version of our `my-ee-demo` Execution Environment Image:

```
[student@ansible-1 ee-collections]$ ansible-builder build --tag my-ee-demo:v2.0
Running command:
  podman build -f context/Containerfile -t my-ee-demo:v2.0 context
Complete! The build context can be found at: /home/student/ee-collections/context
```

Then, review the brand new Execution Environment Image, with two commands `podman images` and `ansible-navigator images`

```shell
[student@ansible-1 ee-collections]$ podman images
REPOSITORY                                                            TAG         IMAGE ID      CREATED             SIZE
localhost/my-ee-demo                                                  v2.0        618973a5a017  About a minute ago  436 MB
<none>                                                                <none>      69f55e66b494  2 minutes ago       593 MB
<none>                                                                <none>      4c48be3b4640  4 minutes ago       315 MB
<none>                                                                <none>      fad1d1b26079  9 minutes ago       232 MB
<none>                                                                <none>      374a00032567  9 minutes ago       318 MB
<none>                                                                <none>      fc5c7be36fde  About an hour ago   232 MB
<none>                                                                <none>      63ad9b5d08fa  About an hour ago   318 MB
<none>                                                                <none>      e488ea4e6207  About an hour ago   232 MB
<none>                                                                <none>      e4396223872b  About an hour ago   318 MB
<none>                                                                <none>      93df4ee04fc7  About an hour ago   306 MB
localhost/my-ee-demo                                                  v1.0        5a63e5230497  3 hours ago         452 MB
<none>                                                                <none>      d9d839ff7a32  3 hours ago         485 MB
<none>                                                                <none>      59cd06e0aa9f  3 hours ago         339 MB
<none>                                                                <none>      cdf36a0f164b  23 hours ago        253 MB
<none>                                                                <none>      439c692fca7b  23 hours ago        339 MB
<none>                                                                <none>      433c8492ade2  23 hours ago        253 MB
<none>                                                                <none>      af80478bb0db  23 hours ago        339 MB
<none>                                                                <none>      79575353b239  23 hours ago        253 MB
<none>                                                                <none>      75ac926d2a3a  23 hours ago        339 MB
registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8  1.0         36a5c93b87dd  13 days ago         1.68 GB
registry.redhat.io/ansible-automation-platform-23/ee-29-rhel8         1.0         58c796a146e3  13 days ago         888 MB
registry.redhat.io/ansible-automation-platform/ee-minimal-rhel8       2.14        ab66adc81863  3 weeks ago         306 MB
registry.redhat.io/ansible-automation-platform/ansible-builder-rhel8  1.2         fa3b605f379b  6 weeks ago         220 MB
registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8  1.0.0-208   a36109621782  6 months ago        1.66 GB
```

```shell
[student@ansible-1 ee-collections]$ ansible-navigator images
  Image                                                                       Tag                                Execution environment                                                       Created                                         Size
0│ansible-builder-rhel8                                                       1.2                                False                                                                       6 weeks ago                                     220 MB
1│ee-29-rhel8                                                                 1.0                                True                                                                        13 days ago                                     888 MB
2│ee-minimal-rhel8                                                            2.14                               True                                                                        3 weeks ago                                     306 MB
3│ee-supported-rhel8                                                          1.0                                True                                                                        13 days ago                                     1.68 GB
4│ee-supported-rhel8                                                          1.0.0-208                          True                                                                        6 months ago                                    1.66 GB
5│my-ee-demo                                                                  v2.0                               True                                                                        2 minutes ago                                   436 MB
6│my-ee-demo                                                                  v1.0                               True                                                                        3 hours ago                                     452 MB



^b/PgUp page up                                ^f/PgDn page down                                ↑↓ scroll                                esc back                                [0-9] goto                                :help help
```

Then, press `**5**` to select `my-ee-demo:v2.0` and then press `**2**` to select `Ansible version and collections` and confirm that the `collections`, `python packages`, and `system packages` that we defined, exists on the Execution Environment Image

```
 0│---
 1│ansible:
 2│  collections:
 3│    details:
 4│      ansible.netcommon: 5.1.1
 5│      ansible.posix: 1.5.4
 6│      ansible.utils: 2.10.3
 7│      community.crypto: 2.14.0
 8│      newswangerd.collection_demo: 1.0.11
 9│  version:
10│    details: ansible [core 2.14.6]
```

That is all for now, let's jump over the next section.


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
[Previous Exercise](../1.7-role) - [Next Exercise](../3.2-collections)

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
