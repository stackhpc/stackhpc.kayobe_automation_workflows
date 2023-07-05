Kayobe Automation Workflows (GitHub)
=========

This Ansible role is capable of generating GitHub workflow files for performing CI/CD related activities with OpenStack via Kayobe.
See the table below for a full list of all the currently supported kayobe automation tasks.

| **Name** | **Description** |
|:---:|:---:|
| **build-kayobe-docker-image** | Build a new kayobe docker image whenever a new tag is pushed to the repository. The resulting image is then pushed to a docker registry such as [ghcr.io](http://ghcr.io). |
| **run-config-diff** | When a pull request is opened generate diff showing the changes made to the configuration. |
| **run-infra-vm-host-package-update** | Perform a package update of the infra VMs hosts on demand. |
| **run-infra-vm-host-configure** | Perform a host configure of the infra VMs on demand. |
| **run-infra-vm-provision** | Provision infra VMs on demand. |
| **run-infra-vm-service-deploy** | Perform a service deploy against infra VMs on demand. |
| **run-network-connectivity-check** | Execute a network connectivity check to ensure all hosts are reachable and can reach `nc_external_ip ` & `nc_external_hostname`. |
| **run-overcloud-container-image-pull** | Pull container images from a container registry. |
| **run-overcloud-database-backup** | Perform a backup of the database used by the overcloud. |
| **run-overcloud-host-configure** | Perform an overcloud host configure. |
| **run-overcloud-host-package-update** | Update system packages on the overcloud hosts. |
| **run-overcloud-inventory-discover** | Get an inventory of nodes. |
| **run-overcloud-provision** | Provision overcloud nodes. |
| **run-overcloud-service-deploy** | Deploy overcloud services. |
| **run-overcloud-service-reconfigure** | Reconfigure services across the overcloud. |
| **run-overcloud-service-upgrade** | Perform an upgrade of overcloud services. |
| **run-seed-host-configure** | Configure the seed host. |
| **run-seed-host-package-update** | Update the system packages of the seed host. |
| **run-seed-hypervisor-host-configure** | Configure the seed hypervisor host. |
| **run-seed-hypervisor-host-package-update** | Perform a package update of the seed hypervisor host. |
| **run-seed-service-deploy** | Deploy services on the seed. |
| **run-seed-vm-provision** | Provision the seed VM. |
| **run-tempest** | Perform tests against the deployed openstack environment with tempest. |

Role Variables
--------------

The following variables can be used to make small adjustments to the composition of the workflows.

`output_directory`: control the location where the workflows shall be written to.

`runner_name`: name of the GitHub runner used by the workflows see [runs-on](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idruns-on).

`image_url`: full URL of the kayobe container image complete with registry and tag.

`registry_password`: password used to authenticate with the docker registry.

`kayobe_arguments`: a dictionary of arguments that can be used to override the default arguments found within `vars/main.yml`. For example if you wanted to change the value of `KAYOBE_ENVIRONMENT` from its default of `production` you can simply add `KAYOBE_ENVIRONMENT` to this dictionary and it will take precedence over the defaults.

If you wish to make more impactful changes such as which workflows are built and what they contain then see the list of dictionaries called `workflows` in `defaults/main.yml`

`workflows:` is a list of dictionaries that contains each of the workflows described above. A given list element is made up of the following:

- `name`: the name which the workflow shall refer to itself as within GitHub workflows user interface.

- `file_name`: name which the workflow will be saved as.

- `trigger`: a dictionary of event triggers that can be used to start the workflow.

- `arguments`: list of arguments keys used by the automation task the contents will be acquired from `kayobe_arguments` or the defaults.

The following will override `workflows` to ensure only `Run overcloud database backup` is generated.

```yaml
workflows:
  - name: Run overcloud database backup
    file_name: run-overcloud-database-backup.yml
    trigger:
      workflow_dispatch: *combined_inputs
      schedule:
        cron: "30 0 * * *"
    arguments:
      - KOLLA_TAGS
      - KOLLA_LIMIT
      - KAYOBE_TAGS
      - KAYOBE_LIMIT
      - HOME
```

Example Playbook
----------------

The following example playbook will generate a series of `reference` workflows which can be found under `.github/workflows`

```yaml
- name: Write Kayobe Automation Workflows
  hosts: localhost
  collections:
    - stackhpc.kayobe_automation_workflows
```

Hooks [Experimental]
--------------------

> :warning: This feature is marked as `experimental` at the moment as it is not clear how to configure `ansible.builtin.template` to look at the `${{ playbook_dir }}/templates/hooks/`.

Workflows can be expanded with the use of `hooks` which are templates that if provided can be inserted into the appropriate location enabling the introduction of additional steps within the workflow job.
This could include the use of HashiCorp Vault or installing and configuring a network proxy.



```yaml

- name: Import secrets via Hashicorp Vault
  id: secrets
  uses: hashicorp/vault-action@v2.5.0
  with:
    url: https://vault.stackhpc.com:8200
    method: approle
    roleId:  ${{ secrets.ROLE_ID }}
    secretId: ${{ secrets.SECRET_ID }}
    tlsSkipVerify: true
    secrets: |
      stackhpc/data/github kayobe_vault_password_${{ needs.env.outputs.environment }} | KAYOBE_VAULT_PASSWORD ;
      stackhpc/data/github kayobe_automation_ssh_private_key_${{ needs.env.outputs.environment }} | KAYOBE_AUTOMATION_SSH_PRIVATE_KEY ;


```

License
-------

Apache License 2.0

Author Information
------------------

[StackHPC](https://www.stackhpc.com/)
