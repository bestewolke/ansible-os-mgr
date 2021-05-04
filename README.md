# ansible-os-mgr
Manage OpenStack resources with Ansible

This repository contains example Playbooks and Roles how to manage your OpenStack resources.  

The ECS delete/start/stop Playbooks will retrieve your current VMs directly from OpenStack.  
So you always have the latest state and there is no need to manually keep track of your instances.

## Requirements
- Ansible 2.10 or newer
- openstacksdk
- openstack.cloud collection (install it with `ansible-galaxy collection install -r requirements.yaml`)

## clouds.yaml
Authentication towards OpenStack is driven by the `clouds.yaml` file.  
You can specify which cloud to use from there in the [`my_os_cloud.yaml`](my_os_cloud.yaml) file:
```yaml
# Enter the name of your cloud to use from clouds.yaml
cloud: open-telekom-cloud
```

# Playbooks

## ECS (Elastic Cloud Server / Virtual Machine)

### Create
The [`ecs_create.yaml`](ecs_create.yaml) playbook creates new VMs.

#### Variables in [`roles/ecs_create/vars/main.yaml`](roles/ecs_create/vars/main.yaml):
One thing you definitely have to change is the `key_name` variable to your own key name.
```yaml
availability_zone: eu-de-03
image: Standard_Debian_10_latest
volume_size: 5
key_name: KeyPair-Sebastian
flavor: s3.medium.1
```

#### Number of instances to deploy:
You can adjust the number of VMs to create in [`ecs_create.yaml`](ecs_create.yaml), the example is set to two:
```yaml
loop: "{{ range(0, 2)|list }}"
```

### Delete
The [`ecs_delete.yaml`](ecs_delete.yaml) playbook deletes VMs.   
The default configuration will wipe all VMs in the current project. But you can work with filters to only delete certain VMs.  
Those can be set directly in the role [`roles/ecs_delete/tasks/main.yaml`](roles/ecs_delete/tasks/main.yaml):
```yaml
# Get only stopped instances named <test*> and with the tag "my=tag"
- openstack.cloud.server_info:
    cloud: "{{ cloud }}"
    server: test*
    filters:
      vm_state: stopped
      tags: ["my=tag"]
  register: servers
```

For maximum speed the delete requests get sent as "fire and forget" in `async` mode with `poll: 0`. So you will only see "changed" as the task status.  
As an example: For 500 VMs it takes approximately 30 minutes to delete them (around 10 minutes to get the instances from OpenStack and around 20 minutes to send the delete requests).

### Start
The [`ecs_start.yaml`](ecs_start.yaml) playbook starts (only stopped) VMs.  
This will retrieve all currently stopped VMs from OpenStack and start those.  
Further filtering can be applied in [`roles/ecs_start/tasks/main.yaml`](roles/ecs_start/tasks/main.yaml) the same way as in the ECS [delete](#delete) example.

For maximum speed the start requests get sent as "fire and forget" in `async` mode with `poll: 0`. So you will only see "changed" as the task status.

### Stop
The [`ecs_stop.yaml`](ecs_stop.yaml) playbook stops (only running) VMs.  
This will retrieve all currently running VMs from OpenStack and stop those.  
Further filtering can be applied in [`roles/ecs_stop/tasks/main.yaml`](roles/ecs_stop/tasks/main.yaml) the same way as in the ECS [delete](#delete) example.

For maximum speed the stop requests get sent as "fire and forget" in `async` mode with `poll: 0`. So you will only see "changed" as the task status.

## Security Group
The [`sg_create.yaml`](sg_create.yaml) playbook creates a security group for SSH access.  
You need to run this before creating your first VM as the security group is used during VM creation.

## VPC (Network, Subnet, Router)
The [`vpc_create.yaml`](vpc_create.yaml) playbook provisions a network, subnet & router.  
You need to run this before creating your first VM as the network is used during VM creation.
