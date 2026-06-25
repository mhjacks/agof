## init_env

This folder contains the Ansible code to initialize the deployment environment for AGOF.

For AWS deployments, [main.yml](main.yml) validates the bootstrap target and imports [aws/main.yml](aws/main.yml), which:

- Looks up the latest Red Hat RHEL marketplace AMI in the target region (or uses a pinned AMI from the vault)
- Provisions VPC, subnet, security group, and EC2 instances
- Registers each node with Red Hat Subscription Manager
- Installs bootstrap packages and creates the `aap` user for the containerized AAP install

Configuration variables live in [vars/bootstrap_vars.yml](vars/bootstrap_vars.yml). Secrets are supplied via `~/agof_vault.yml`.

The full install flow invokes this playbook from [site.yml](../site.yml) as Phase 1.
