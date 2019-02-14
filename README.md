# Diverse Heat templates for Ecosystem infrastructure

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
