# AWX

## Prepare Software-config image

Templates may use SoftwareConfig and SoftwareDeployment resources. It gives a benefit of avality to perform runtime configuration of the servers without recreating them (as with using user_data). However this require specially prepared image. It can be done as follows:
```
  sudo pip install git+https://git.openstack.org/openstack/diskimage-builder.git
  git clone https://git.openstack.org/openstack/tripleo-image-elements.git
  git clone https://git.openstack.org/openstack/heat-agents.git
  export ELEMENTS_PATH=tripleo-image-elements/elements:heat-agents/
  disk-image-create vm \
    fedora selinux-permissive \
    os-collect-config \
    os-refresh-config \
    os-apply-config \
    heat-config \
    heat-config-ansible \
    heat-config-cfn-init \
    heat-config-docker-compose \
    heat-config-kubelet \
    heat-config-puppet \
    heat-config-salt \
    heat-config-script \
    -o fedora-software-config.qcow2
  openstack image create --disk-format qcow2 --container-format bare --min-disk 6 --min-ram 512 fedora-software-config < \
    fedora-software-config.qcow2
```

## Provision infrastructure

In order to provision the infrastructure for the AWX stack a heat template is used. It can be created following this steps:

* prepare software-config enabled image and upload it to the cloud
* Prepare SSH Keypairs for accessing bastion and AWX server
* modify `awx/awx.env.yaml` with appropriate values
* execute `openstack-3 stack create -e awx.env.yaml -t awx_stack.yaml awx_stack`
* updated should enable rollback, so execute `openstack-3 stack update -e awx.env.yaml -t awx_stack.yaml --rollback enabled awx_stack`
* assign EIP to LB
* Provision awx_ssh_pub_key (output of the stack) into the git, so that awx can pull
* enable postgresql DB access into the VPC (VPC peering)
* install postgresql on some host in the VPC (i.e. awx)
*  dabatase can be created with following commands:
```
    $ psql -h __HOST_OF_PG__ -d postgres -U root
    $ create database awx;
    $ create user awx with encrypted password '__SOME_PWD__';
    $ grant all privileges on database awx to awx;
```
* on the awx host place `~root/awx_inventory_vars.yaml` file with diverse data (consumed by awx role from https://github.com/opentelekomcloud-infra/ansible-awx) and provision ssl certificates into /etc/nginx/ssl (referred by name from `awx_inventory_vars.yaml`)
```
---
pg_hostname: HOST
pg_username: USER
pg_password: PWD
pg_database: DB
pg_port: 5432

awx_admin_password: INITIAL_ADMIN_PASSWORD
awx_secret_key: SECRET_KEY

server_url: MY_DOMAIN
vhost_name: awx
ssl_certificate: CERTIFICATE_BASE_NAME
ssl_certificate_key: KEY_BASE_NAME
```
NOTE: if this is part of SoftwareDeployment, then secrets are visible to anyone with project access


Remaining provisioning would be pulled automatically by the AWX server from the repository mentioned in the env


Ideally this template can be invoked from the ansible-awx repo by ansible to bring stuff together

After stack is created and the first ansible-pull completed the AWX installation can be started manually (TODO: automate this also):

```
  (root@awx) $ cd ~/workspace/awx/installer
  (root@awx) $ ansible-playbook -i ~/awx_inventory install.yml
```

## TODO

* Automate this properly to the end
* Clarify how to deploy secrets and certificates to awx server
