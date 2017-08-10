ssh_config_from_inventory
=========

Manage ssh client configuration based on Ansible inventory.

Requirements
------------

This role requires the `debug`, `template`, and `blockinfile` module. The `blockinfile` was added in Ansible version 2.0 so that is the minimum supported version.

Role Variables
--------------

### User Defined Variables

|Variable|Default|Description|
|---|---|---|
| `ssh_configs_dir` | `{{ playbook_dir }}/ssh_configs` | Location where the group specific SSH config files are stored prior to merging to a single SSH config file. |
| `ssh_config_file` | `{{ ssh_configs_dir }}/ssh_config` | Where should the SSH client configuration be written to? Most implementations use ``~/.ssh/config` so you can change this if you want. |
| `inventory_groups` | `["all"]` | Which inventory groups should we read to create SSH client configuration for? By default the built-in group `all` will be used since it should always be valid. |

### Role Consumed Variables
|Variable|Usage|Description|
|---|---|---|
|`inventory_hostname_short`|Required but usually set automatically.|This name will be used to define the `Host` section of the SSH client configuration.|
|`ansible_host`|optional|If set, a `HostName` value will be set for the host. This is useful when you do not have the host in DNS.|
|`ansible_port`|optional|If set, a `Port` value will be set for the host.|
|`ansible_user`|optional|If set, a `User` value will be set for the host.|
|`ansible_ssh_private_key_file`|optional|If set, a `IdentityFile` value will be set for the host.

All Role Consumed Variables can be defined in your [Ansible inventory file](http://docs.ansible.com/ansible/latest/intro_inventory.html).

Dependencies
------------

No other roles are required for this to work.

Example Playbook
----------------

Here is an example play that you may wish to add to one of your existing playbooks. All the variables are defined here for illustration purposes. The role is intended to be used by localhost as the current user, though you can change that to suit your needs.
    - hosts: localhost
      become: "False"
      gather_facts: "False"
      vars:
        inventory_groups: ["production","staging"]
        ssh_configs_dir: "/tmp/ssh_config"
        ssh_config_file: "{{ ssh_configs_dir }}/ssh_config"
      roles:
        - russStarr.ssh_config_from_inventory

License
-------

Apache 2.0

Author Information
------------------

N/A
