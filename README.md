[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

# Ansible GitOps Framework (AGOF)
<!-- vscode-markdown-toc -->
* 1. [How to Use It](#HowtoUseIt)
	* 1.1. [Installation](#Installation)
	* 1.2. [Uninstallation](#Uninstallation)
* 2. [Entry Points](#EntryPoints)
* 3. [Using the Containerized Installer vs. The traditional one](#UsingtheContainerizedInstallervs.Thetraditionalone)
	* 3.1. ["API" Install (aka "Bare")](#APIInstallakaBare)
	* 3.2. ["From OS" Install](#FromOSInstall)
	* 3.3. [Default Install](#DefaultInstall)
	* 3.4. [Convenience Features Install (defined by options in the "default" install)](#ConvenienceFeaturesInstalldefinedbyoptionsinthedefaultinstall)
* 4. [agof_vault.yml Configuration](#agof_vault.ymlConfiguration)
	* 4.1. [Common Configuration](#CommonConfiguration)
	* 4.2. [Initialization Environment Configuration](#InitializationEnvironmentConfiguration)
	* 4.3. [Automation Hub Specific Configuration](#AutomationHubSpecificConfiguration)
	* 4.4. [AWS-Specific Configuration](#AWS-SpecificConfiguration)
	* 4.5. [ImageBuilder-Specific Configuration](#ImageBuilder-SpecificConfiguration)
* 5. [What the Framework Does, Step-by-Step](#WhattheFrameworkDoesStep-by-Step)
	* 5.1. [Pre-GitOps Steps](#Pre-GitOpsSteps)
		* 5.1.1. [[Pre-init](init_env/pre_init_env.yml) (mandatory)](#Pre-initinit_envpre_init_env.ymlmandatory)
		* 5.1.2. [[Environment Initialization](init_env/main.yml) (optional; enabled by default; `make install` entry point)](#EnvironmentInitializationinit_envmain.ymloptionalenabledbydefaultmakeinstallentrypoint)
		* 5.1.3. [[Containerized Install](containerized_install/main.yml) (`make install` default)](#ContainerizedInstallcontainerized_installmain.ymlmakeinstalldefault)
		* 5.1.4. [OpenShift Targets](#OpenShiftTargets)
	* 5.2. [GitOps Step](#GitOpsStep)
* 6. [Acknowledgements](#Acknowledgements)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->

This repository is part of Red Hat's [Validated Patterns](https://validatedpatterns.io) effort, which in general
aims to provide Reference Architectures that can be run through CI/CD systems.

The goal of this project in particular is to provide an extensible framework to do GitOps with Ansible Automation
Platform, and as such to provide useful facilities for developing Patterns (community and validated) that function with Ansible Automation Platform as the GitOps engine.

The thinking behind this effort is documented in the [Ansible Pattern Theory](https://github.com/mhjacks/ansible-pattern-theory) repository.

##  1. <a name='HowtoUseIt'></a>How to Use It

The default installation will provide an AAP 2.5 installation deployed via the Containerized Installer, with services deployed this way:

| URL Pattern | Service |
|-------------|---------|
| https://aapnode.fqdn3/ | Controller API |

Automation Hub and EDA are enabled by default, but they can be turned off if desired.

By default, the framework will apply license content specified by the `manifest_content` variable, but will not further configure Controller or Automation Hub beyond the defaults.

From there, a minimal example pattern is available to download and run [here](https://github.com/validatedpatterns-demos/agof_minimal_config.git). To use this example, set the following variables in your `agof_vault.yml`.
`agof_statedir` is where the config repo will be checked out by the process. Any repo that can be used with the controller_configuration collection can be used as the `agof_iac_repo`.

```yaml
agof_statedir: "{{ '~/agof' | expanduser }}"
agof_iac_repo: "https://github.com/validatedpatterns-demos/agof_minimal_config.git"
```

###  1.1. <a name='Installation'></a>Installation

```shell
./pattern.sh make install
```

This builds the default pattern configuration on AWS, which (by default) includes a containerized install of AAP 2.5 on a single AWS VM. Various add-ons can be included by adding variables to the `~/agof_vault.yml` file as described below - these options will all be honored as the pattern installs itself.

###  1.2. <a name='Uninstallation'></a>Uninstallation

```shell
./pattern.sh make aws_uninstall
```

This destroys all of the AWS infrastructure built for the pattern - starting with the VPC it creates for the pattern and all of the resources attached to it.

##  2. <a name='EntryPoints'></a>Entry Points

This is a framework for building Validated Patterns that use Ansible Automation Platform (AAP) as their underlying GitOps engine. To that end, the framework has these deployment models (in increasing level of complexity):

##  3. <a name='UsingtheContainerizedInstallervs.Thetraditionalone'></a>Using the Containerized Installer vs. The traditional one

In AAP 2.5, the containerized install feature has reached General Availability and is the preferred way to install AAP on non-OpenShift environments.

###  3.1. <a name='APIInstallakaBare'></a>"API" Install (aka "Bare")

```shell
./pattern.sh make api_install
```

In this model, you provide an (already provisioned) AAP endpoint. It does not need to be entitled, it just needs to be running the AAP Controller. You supply the manifest contents, endpoint hostname, admin username (defaults to "admin"), and admin password, and then the installation hands off to a `agof_controller_config_dir` you define. This is provided for users who have their own AAP installations on bare metal or on-prem or do not want to run on AWS. It is also useful in situations where the AAP deployment topology is more complex than what we provide in the pattern provisioner.

###  3.2. <a name='FromOSInstall'></a>"From OS" Install

```shell
./pattern.sh make from_os_install INVENTORY=(your_inventory_file)
```

In this model, you provide an inventory file with a fresh RHEL 9 installation. The workflow will use the containerized installer to install AAP on the host
in the aap_controllers group. (Many other topologies are possible with the AAP installation framework; see the Ansible Automation Platform Planning and Installation guides for details.) If you need to install a pattern on a cluster with a different topology than this, use the API install mechanism. This mechanism will run a (slightly) opinionated install of the AAP and Hub components, and will add some conveniences like default execution environments and credentials. Like the "API" install, the install will then be handed over to a `agof_controller_config_dir` you define.

The reason for making this a separate option is to make it easy for those who are not used to installing AAP to get up and running with it given a couple of VMs (or baremetal instances). Requirements for this mode are as follows:

* Must be running a version of RHEL that AAP supports
* Must be properly entitled with a RHEL subscription

It is not possible to test all possible scenarios in this mode, and we do not try.

Note that INVENTORY defaults to '~/inventory_agof' if you do not specify one.

Your inventory *must* define an `aap_controllers` group (which will be configured as the AAP node) and an `automation_hub` group which will be configured as the automation hub, if you want one. You must also specify `username` if you want it to be something besides the default 'ec2-user' (which it does not create or otherwise manage). Similarly, you should set `controller_hostname` (including the 'https://' schema) because that will default to something AWS-specific otherwise. As with all Ansible inventory files, you can set other variables here and the plays will use them.

Example `~/inventory_agof` (for just AAP, which is the default):

```ini
[build_control]
localhost

[aap_controllers]
192.168.5.207

[automation_hub]

[eda_controllers]

[aap_controllers:vars]

[automation_hub:vars]

[all:vars]
ansible_user=myuser
ansible_ssh_pass=mypass
ansible_become_pass=mypass
ansible_remote_tmp=/tmp/.ansible
username=myuser
aap_hostname=192.168.5.207
```

###  3.3. <a name='DefaultInstall'></a>Default Install

```shell
./pattern.sh make install
```

In this model, you provide AWS credentials in addition to the other components needed in the "bare" install. The framework will build an AWS image using Red Hat's ImageBuilder, deploy that image onto a new AWS VPC and subnet, and deploy AAP on that image using the command line installer. It will then hand over the configuration of the AAP installation to the specified `agof_controller_config_dir`.

###  3.4. <a name='ConvenienceFeaturesInstalldefinedbyoptionsinthedefaultinstall'></a>Convenience Features Install (defined by options in the "default" install)

```shell
./pattern.sh make install
```

This model builds on the Default installation by adding the options for building extra VMs besides AAP - the configured VM options include Private Automation Hub (hub), Identity Management (idm), and Satellite (satellite). The framework will configure Automation Hub in this mode, and configure AAP to use it; it will not configure idm or satellite beyond what they need to be managed by AAP. (The configuration of idm and satellite is the subject of the first pattern to use this framework.)

The variables to include builds for the extra components are all Ansible booleans that can be included in your configuration:

`automation_hub`
`eda`

##  4. <a name='agof_vault.ymlConfiguration'></a>agof_vault.yml Configuration

###  4.1. <a name='CommonConfiguration'></a>Common Configuration

| Name                      | Description                          | Required | Default  | Notes |
| ------------------------- | ------------------------------------ | -------- | -------- | ------ |
| admin_user                | Admin User (for AAP and/or Hub)      | false     | 'admin' |        |
| admin_password            | Admin Password (for AAP and/or Hub)  | true    |           |        |
| redhat_username           | Red Hat Subscriber Username          | true    |           |        |
| redhat_password           | Red Hat Subscriber Password          | true    |           |        |
| redhat_registry_username_vault  | Red Hat Subscriber Username    | true    |           |  |
| redhat_registry_password_vault  | Red Hat Subscriber Password    | true    |           |  |
| manifest_content          | Base64 encoded Manifest to Entitle   | true    |           | Can be loaded directly from a file using a construct like this: `"{{ lookup('file', '~/Downloads/manifest.zip') | b64encode }}"` |
| automation_hub_certified_url | URL for Certified Content  | false    | <https://console.redhat.com/api/automation-hub/content/published/>   | This refers to the automation hub section on [https://console.redhat.com](https://console.redhat.com).  It is the endpoint that is used to download Validated Content in addition to any public Galaxy content needed |
| automation_hub_validated_url | URL for Validated Content  | false    | <https://console.redhat.com/api/automation-hub/content/validated/> | This refers to the automation hub section on [https://console.redhat.com](https://console.redhat.com).  It is the endpoint that is used to download Certified Content in addition to any public Galaxy content needed |
| automation_hub_token_vault| Subscriber-specific token for Content | true    |                    |
| automation_hub            | Flag to build and enable Automation Hub | false     | false |  | Building a Private Automation Hub is necessary if your pattern builds an Execution Environment that is not hosted on a public container registry. |
| eda            | Flag to build and enable Event Driven Automation controller | false     | true |  |  |
| agof_controller_config_dir    | Directory to pass to controller_configuration | false  |  | This directory is the key one to load all other AAP Controller configuration. The framework will populate it by checking out the `agof_iac_repo` version `agof_iac_repo_version` (default: `main`) |
| controller_launch_jobs    | List of jobs to run after controller_configuration has run | false     |  | Use this to start a job (or jobs) that do not have aggressive schedules, and that are ready to run as soon as the controller is configured. The fewer jobs listed here the better. |

###  4.2. <a name='InitializationEnvironmentConfiguration'></a>Initialization Environment Configuration

| Name                      | Description                          | Required | Default  | Notes |
| ------------------------- | ------------------------------------ | -------- | -------- | ------------------ |
| init_env_collection_install | Whether to install collections required by the framework | false | true |  |
| init_env_collection_install_force | Whether to use the `force` argument when installing collections | false | false | Forces the installation of declared dependencies if true |
| special_collection_installs | "Bundled" collection installations (references files in repodir) | false | `[]` | A mechanism to allow the installation of collections bundled into the pattern, if the ones published in galaxy and/or Automation Hub are not sufficient |
| offline_token             | Red Hat Offline Token                | false    |          | Used to build the imagebuilder image |

###  4.3. <a name='AutomationHubSpecificConfiguration'></a>Automation Hub Specific Configuration

| Name                      | Description                          | Required | Default            | Notes |
| ------------------------- | ------------------------------------ | -------- | ------------------ | ------ |
| aap_admin_username        | Admin User (for AAP)                 | false    | '{{ admin_user }}' | |
| aap_admin_password        | Admin Password (for AAP)             | true     | '{{ admin_password }} ' | |
| private_hub_username        | Admin User (for Automation Hub)    | true     | '{{ admin_user }}' | |
| private_hub_password        | Admin Password (for Automation Hub) |true     | '{{ admin_password }} ' | |
| custom_execution_environments | Array of Execution Environments to build on Hub | true     |  `[]`  | The execution environments will be built on the hub node, and then pushed into the hub registry from there. |

###  4.4. <a name='AWS-SpecificConfiguration'></a>AWS-Specific Configuration

| Name                      | Description                          | Required | Default            | Notes  |
| ------------------------- | ------------------------------------ | -------- | ------------------ | ------ |
| aws_account_nbr_vault     | AWS Account Number                   | false    |                    | The AWS Account Number is used by ImageBuilder to share the resulting image to as an AMI |
| aws_access_key_vault      | AWS Access Key String                | false    |                    | The AWS Access Key - can be included if other pattern elements cannot read AWS credentials directly for subsequent pattern use |
| aws_secret_key_vault      | AWS Secret Key String                | false    |                    | The AWS Secret Key - like the access key, can be added if other pattern elements need it |
| ec2_region                | EC2 region to use for builds         | false    |                   | us-east-1 is a reasonable value to use because this is where imagebuilder puts images by default |
| ec2_name_prefix           | Text to add for EC2                  | false    |                   | This is a name to disambiguate your pattern from anything else that might be running in your AWS account. A VPC, subnet, and security group is built from this, and the prefix is added to the `pattern_dns_zone` by default. Additionally, the SSH private key is stored locally in `~/{{ ec2_name_prefix }}`. |
| pattern_dns_zone          | Zone to use for route53 updates      | false    |                   | Definitely set this if doing DNS updates |
| ec2_instances_xtra        | Dictionary of additional ec2_instances to build  | false    |          | Build additional VMs as part of the pattern |

###  4.5. <a name='ImageBuilder-SpecificConfiguration'></a>ImageBuilder-Specific Configuration

*Note:* If you are providing an AMI via the `imagebuilder_ami` variable as opposed to building one from console.redhat.com for the pattern, the pattern still assumes that the RHEL instance will be entitled and will be running a suitable version of RHEL.

| Name                      | Description                          | Required | Default            | Notes |
| ------------------------- | ------------------------------------ | -------- | ------------------ | ------- |
| org_number_vault          | Red Hat Subscriber Organization Number | true  |  | This is the organization number associated with the RHEL instances you need to entitle |
| activation_key_vault      | Activation Key Name to embed in image  | true  |                    | This is an activation key for the Red Hat CDN. It is expected to be able to enable both the base RHEL repos and the AAP repos. |
| skip_imagebuilder_build   | Flag to skip imagebuilder build (also set `imagebuilder_ami` if true) |  false | false     | |
| imagebuilder_ami   | AMI to use for VM creation in AWS  | false             | | It is very possible to re-use another imagebuilder build from a previous installation of the pattern framework, and saves ~15 minutes on a new pattern install to re-use such an image. |
| ami_source_region   | Source Region to copy AMI from if not present in target region  | false             | us-east-1 | Imagebuilder puts its images in us-east-1 by default, but the framework can copy an image built there to another region if you specify one here. If you specify a non-imagebuilder ami and it "starts out" in a different region, you can specify that here. |

##  5. <a name='WhattheFrameworkDoesStep-by-Step'></a>What the Framework Does, Step-by-Step

###  5.1. <a name='Pre-GitOpsSteps'></a>Pre-GitOps Steps

####  5.1.1. <a name='Pre-initinit_envpre_init_env.ymlmandatory'></a>[Pre-init](init_env/pre_init_env.yml) (mandatory)

Pre-initialization, for our purposes, refers to things that have to be installed on the installation workstation/provisioner node to set up the Ansible environment to deploy the rest of the pattern.

##### Build ansible.cfg

Because ansible.cfg needs to be configured with the automation hub endpooint, we generate it from a template. The ansible.cfg includes endpoints for both public and Automation Hub galaxy servers and a vault password path (for `ansible-vault`). The template used is [here](init_env/templates/ansible.cfg.j2). The automation hub token is read from the vault and injected into the environment for the collection installation phase.

##### Collection dependency install

The next thing the pre-init play does is install dependency collections based on what's contained in [requirements.yml](requirements.yml). If there are any "special" collections contained in the root of the framework they can also be installed at this time. When the framework was under development this was necessary with redhat.satellite but the necessary changes have been upstreamed as of 3.11; 3.12 has been released since then, so this mechanism is no longer needed but is left in as it might be necessary in the future.

####  5.1.2. <a name='EnvironmentInitializationinit_envmain.ymloptionalenabledbydefaultmakeinstallentrypoint'></a>[Environment Initialization](init_env/main.yml) (optional; enabled by default; `make install` entry point)

Environment initialization includes the steps necessary to build the environment (VMs) to install AAP and related tooling; currently the framework can do this on AWS. This step is optional if you wish to provide either an AAP controller API endpoint OR else bring your own inventory/VMs to install AAP Controller and (optionally) Automation Hub on.

##### [image build (optional)](buildimage/main.yml) (optional)

This play builds an image using the Red Hat Console's imagebuilder service; the end result of this process is an image that will serve as an AMI in AWS. (ImageBuilder can build other types of images as well.) The key aspects of this image are that they include the cloud-init package (which helps with certain aspects of initialization), but more importantly, they include the organization number and activation key so that images that are instantiated with this AMI are automatically registered and enabled to install content via the Red Hat CDN.

These images tend to be fairly static, so it is not necessary to build a brand new image every time you run the pattern. You can save the AMI from a previous pattern run, and re-use it as long as you like. Building and uploading the image to AWS represents about 15 minutes of the runtime of the pattern installation.

##### [Initialize the Environment (for AWS)](init_env/aws/main.yml) (optional)

This play handles all the AWS-specific setup necessary to run AAP, (optionally) Automation Hub, (optionally) IdM, (optionally) Satellite, and also offers you the ability to install VMs of your own for a pattern via overrides. The reason for this is that installing the VMs at this stage allows the customer/user to not have to involve AWS credentials in the pattern itself if they do not wish to do so. Of course they are free to include workflows that interact with AWS or other infrastructure if they want - this was done with the intention of simplifying that process if AWS was an implementation detail of the pattern as opposed to the intention of it.

The two key roles that are invoked here are [manage_ec2_infra](init_env/aws/roles/manage_ec2_infra/) and [manage_ec2_instances](init_env/aws/roles/manage_ec2_instances/). Both of these have been adapted from [ansible-workshops](https://github.com/ansible/workshops). The `manage_ec2_infra` role is responsible for setting up the VPC, subnet, and security group for the patten. `manage_ec2_instances` is responsible for actually building the VM instances. It will also manage route53 DNS entries. It is safe to run these roles repeatedly; they will not "double allocate" VMs as long as the VMs are running.

These roles also include code for making hostnames durable across reboots, as well as maintaining `/etc/hosts` on all AWS nodes in the bootstrap set that include all of the other servers in the bootstrap set.

##### [Update Route53 DNS (if needed))](init_env/aws/fix_aws_dns.yml)

The standard public IPs that are offered by AWS (which are used by this framework) are not permanently associated with the VMs. When the VMs cold start, in particular, we can expect IP change events. The VMs themselves are not directly aware of their external IPs. In order to handle this situation, this play can be run from the provisioner node/workstation to update the route53 DNS mappings based on the AWS API.

##### [Teardown AWS Environment)](init_env/aws/teardown.yml) (`make aws_uninstall`)

The teardown play will terminate all VMs associated with a VPC and subnet, and remove the DNS entries managed by the "bootstrap set" (which are just the VMs the framework has been told it has to build).

####  5.1.3. <a name='ContainerizedInstallcontainerized_installmain.ymlmakeinstalldefault'></a>[Containerized Install](containerized_install/main.yml) (`make install` default)

| Name                      | Description                          | Required | Default            | Notes |
| ------------------------- | ------------------------------------ | -------- | ------------------ | ------- |
| containerized_installer_user          | Unprivileged user to create to run AAP | true  | `aap` |  |
| containerized_installer_user_home          | Directory to install containerized AAP into | true  | `/home/{{ containerized_installer_user }}` |  |
| containerized_installer_version  | Minor version of AAP to install  | false  | "2.5" |  |
| automation_hub  | Boolean to indicate whether to install automation_hub | true  | true |  |
| postgresql_admin_username  | Name of postgres admin for services | true  | `postgres` |  |
| postgresql_admin_password  | Password for the postgres user for services | true  | `{{ db_password }}` |  |
| ee_extra_images  | Specifications for extra execution environments to load | false  | `[]` |  |
| de_extra_images  | Specifications for extra decision environments to load (for EDA) | false  | `[]` |  |
| controller_postinstall  | Whether or not to use controller_postinstall feature | false  | `false` |  |
| controller_postinstall_repo_url  | Repository URL for additional controller configuration | false  | |  |
| controller_postinstall_repo_ref  | Tag, branch or SHA in the repo to use for configuration | false  | `main` |  |
| controller_postinstall_dir  | Directory to use to load additional controller config | false  | | Prefer using the `controller_postinstall_repo_url` as it is more flexible |
| hub_admin_password  | Admin user password for automation_hub service | false  | '{{ admin_password }}' |  |
| hub_pg_password  | Postgres password for automation_hub service | false  | '{{ db_password }}' |  |
| hub_postinstall  | Whether to use the `hub_postinstall` feature | false  | false |  |
| hub_postinstall_dir  | Directory to use for hub_postinstall if enabled | false  |  | Prefer hub_postinstall_repo options as they are more flexible  |
| hub_postinstall_repo_url  | Repository to use for hub_postinstall if enabled | false  |  |  |
| hub_postinstall_repo_ref  | Branch, tag, or sha to use in repo for hub postinstall | false  | `main`  |  |
| hub_workers  | How many worker containers to run | false  | `2`  |  |
| hub_collection_signing  | Whether to sign collections automatically | false  | `false`  |  |
| hub_collection_signing_key  | GPG key to sign collections with, if desired | false  |  |  |
| hub_container_signing  | Whether to sign containers automatically | false  | `false`  |  |
| hub_container_signing_key  | GPG key to sign containers (EE's and DE's) with, if desired | false  |  |  |
| eda_admin_password  | EDA Service admin password | false  | `{{ admin_password }}` |  |
| eda_pg_password  | EDA Service postgres password | false  | `{{ db_password }}` |  |
| eda_workers  | Number of EDA workers | false  | `2` |  |
| eda_activation_workers  | Number of EDA activation workers | false  | `2` |  |

First, we run the [Installer Prereqs](containerized_install/roles/installer_prereqs/) role. This sets up a user (default: `aap`) to run AAP as, sets up password-less sudo for that user, and installs the pattern user's `~/.ssh/id_rsa.pub` as an authorized_key for the user. The sudo rule simplifies the process of running the containerized installer, and by default is removed in the cleanup stage. This role also sets `linger` on the AAP user so that services will start at boot time under the AAP user.

Next, we run the [Containerized Installer](containerized_install/roles/installer/) role. This runs the containerized installer as the designated user. This user _cannot_ be root (because of how it uses and configures containerized services). This role downloads and executes the installer, and also places a manifest file (designated by the `manifest_content` variable) as `manifest.zip` to entitle the controller. It runs the containerized installer.

Finally, we run the [cleanup](containerized_install/roles/installer_cleanup/) role if it is enabled (which it is by default). This removed the sudo rule from the AAP user, and removes the manifest.zip file. The installation is now ready for you to use.

##### [AAP Configuration](configure_aap.yml) (mandatory for non-containerized install; `make api_install` entry point)

This play is really the heart and focus of this framework. The rest of the framework exists to facilitate running this play, and providing additional capabilities to the environment in which the play runs. The logic for the play is contained in the [configure_aap](hosts/roles/configure_aap/) role. The `make api_install` entry point uses just the variables related to controller installation as desribed [here](#api-install-aka-bare). Otherwise, it is called inline from the `make install` entry point. It is safe to run multiple times.

###### Entitle AAP

The play uses the same technique and variables to entitle AAP as the role above does; but since we want to preserve the option to call it outside that role, it is duplicated. The key thing is to populate `manifest_contents` with a suitable manifest file.

###### Configure AAP

Using the credentials you supply, the play will now proceed to invoke the `aap_configuration` `dispatch` role on the endpoint based on the configuration found in `controller_configuration_dir` and in your `agof_vault.yml` file. This controls all of the controller configuration. Based on the variables defined in the controller configuration, `dispatch` will call the necessary roles and modules in the right order to configure AAP to run your pattern.

If you need any jobs to be run immediately, then can be specified using the `launch_job` type as part of the AAP
configuration repo.

####  5.1.4. <a name='OpenShiftTargets'></a>OpenShift Targets

Designed to run under OpenShift and configure an OpenShift-hosted AAP instance, AGOF has some special handling for these targets. It is meant to be used by the Validated Patterns aap-config chart, which is meant to run agof_vault.yml

The preinit steps run as they do in other contexts, setting up ansible.cfg and downloading and installing the ansible collections necessary to do the configuration of the AAP instance. The main difference is that inside OpenShift the aap-config provides Kubernetes ConfigMaps and some specific Secrets that can be used to further configure the AAP instance.
Since the OpenShift Validated Patterns experience uses HashiCorp Vault as a secret store, AGOF under OpenShift facilitates using this vault as a secret store.

Additionally, AGOF expects an agof_vault.yml file for any other secret material, which will be included in OpenShift as well. This means that you do not have to use the Vault integration if you would rather not.

When the aap-config Helm values include `aapManifest` (see [mhjacks/aap-config-chart](https://github.com/mhjacks/aap-config-chart) branch `optionalize_secrets`), `pre_init/openshift_vp_preinit.yml` reads the subscription manifest from the **same path the chart mounts** when `aapManifest.source` is `sscsi` (Secrets Store CSI) or `externalSecret` (native Secret volume). If the mount is not available or `aapManifest` is absent, it falls back to the existing `Secret` `aap-config/aap-manifest` API path.

Because the AGOF runner needs predicatability for the existence of the manifest file and the IAC repo and revision, it overrides these specific elements from vault with its own override file, which is created as `~/agof_overrides.yml`. This file is included as the last extra_vars file on the playbook command line. It specifically sets these variables:

* `aap_entitled`: Based on examing the AAP endpoint, whether the instance has been entitled or not
* `admin_user`: Hardcoded to `admin` currently.
* `admin_password`: Has the value of the password randomly generated by the AAP Operator installation
* `aap_hostname`: The endpoint of the AAP instance, as discovered by looking for its route in the ansible-automation-platform namespace
* `agof_iac_repo`: Set by retrieving it from the helm values, so that it overries what may be in your local `agof_vault.yml` file.
* `agof_iac_repo_version`: Set by retrieving it from the Helm chart, as above.
* `controller_license_src_file`: Set by creating a temporary file from the mounted manifest or the manifest Secret
* `secrets`: Special data structure that contains other elements discovered from OpenShift.
* `helm_values`: All of the helm values available to the application.

Additionally, the `agof_vault.yml` file that is passed to Vault will be used as the agof_vault.yml file inside the configuration container, so any variables set here will be usable by the framework; but any of the values listed above will be overridden by the overrides file. The goal is to provide as similar an experience as possible between the different ways of running AGOF.

Meanwhile, the helm values that have been passed into the aap-config application will be available in the `helm_values` variable.

###  5.2. <a name='GitOpsStep'></a>GitOps Step

The framework is only truly in GitOps mode once the configuration has been fully applied to the AAP controller and the AAP controller has taken responsibility for managing the environment. Prior to that point, there are many opportunities (by design) to inject non-declarative elements into the environment. Beyond that point, changes should be made to the environment or the workflows that configure it by pushing git commits to the repositories the pattern uses.

##  6. <a name='Acknowledgements'></a>Acknowledgements

This repository represents an interpretation of GitOps principles, as developed in the Hybrid Cloud Patterns GitOps framework for Kubernetes, and an adaptation and fusion of two previous ongoing efforts at Red Hat: [Ansible-Workshops](https://github.com/ansible/workshops) and [LabBuilder2/RHISbuilder](https://github.com/parmstro/labbuilder2).

The AWS interactions come from Ansible Workshops (with some changes); the IDM and Satellite build components come from LabBuilder2/RHISbuilder. I would also like to specifically thank Mike Savage and Paul Armstrong, co-workers of mine at
Red Hat, who have provided encouragement and advice on how to proceed with various techniques in Ansible in particular.
