ssh_config_from_inventory
=========
![travis_status](https://travis-ci.org/russStarr/ssh_config_from_inventory.svg?branch=master)

Manage ssh client configuration based on Ansible inventory.

Requirements
------------

This role requires the `debug`, `template`, and `blockinfile` module. The `blockinfile` was added in Ansible version 2.0 so that is the minimum supported version.

Installation
------------
By default Ansible Galaxy will install roles in a system wide location at /etc/ansible/roles. If you have sudo privileges and wish to place them there, you can do a:

```
sudo ansible-galaxy install russStarr.ssh_config_from_inventory
```

If you do not have root and only want to install the role in your current Ansible project folder using the default `roles` directory, then issue this command:

```
ansible-galaxy install russStarr.ssh_config_from_inventory --roles-path roles
```

Role Variables
--------------

### User Defined Variables

|Variable|Default|Description|
|---|---|---|
| `ssh_configs_dir` | `{{ playbook_dir }}/ssh_configs` | Location where the group specific SSH config files are stored prior to merging to a single SSH config file. |
| `ssh_config_file` | `{{ ssh_configs_dir }}/ssh_config` | Where should the SSH client configuration be written to? Most implementations use `~/.ssh/config` so you can change this if you want. |
| `inventory_groups` | `["all"]` | Which inventory groups should we read to create SSH client configuration for? By default the built-in group `all` will be used since it should always be valid. |

### Role Consumed Variables
|Variable|Usage|Description|
|---|---|---|
|`inventory_hostname_short`|Required but usually set automatically.|This name will be used to define the `Host` section of the SSH client configuration.|
|`ansible_host`|optional|If set, a `HostName` value will be set for the host. This is useful when you do not have the host in DNS.|
|`ansible_user`|optional|If set, a `User` value will be set for the host.|
|`ansible_port`|optional|If set, a `Port` value will be set for the host.|

All Role Consumed Variables can be defined in your [Ansible inventory file](http://docs.ansible.com/ansible/latest/intro_inventory.html).

Here is an example of an Ansible inventory file making use of all the variables.
```
[production]
server1.domain.tld ansible_host=10.10.10.10
server2.domain.tld
server3.domain.tld

[production:vars]
ansible_user=ansible
ansible_ssh_private_key_file=~/.ssh/production_key.pem
ansible_port=2222
```

Dependencies
------------

No other roles are required for this to work.

Compatibility
-------------
The configuration generated is intended to work with OpenSSH clients. Almost all Linux/BSD Operated Systems are listed as compatible with the assumption that OpenSSH is very portable, especially when it comes to configuration files.

Work Flow
---------
The steps used in the tasks for this role are high-lighted from a high level.

1. First create the folder to store local SSH client configuration files by group.
2. Go through all groups defined in `inventory_groups` and create a local SSH client configuration.
3. Combine all the SSH client configuration files and merge them into the User Defined `ssh_config_file`, which can be ~/.ssh/config for convenience.

> Note: The `blockinfile` module will manage blocks of configuration in the `ssh_config_file`. You can safely have existing configuration in your `ssh_config_file` because `blockinfile` will only look for config blocks as defined be the `{mark}` value.

Example Playbook
----------------

Here is an example play that you may wish to add to one of your existing playbooks. All the variables are defined here for illustration purposes. The role is intended to be used by localhost as the current user, though you can change that to suit your needs.

```
- hosts: localhost
  become: "False"
  gather_facts: "False"
  vars:
    inventory_groups: ["production","staging"]
    ssh_configs_dir: "/tmp/ssh_config"
    ssh_config_file: "{{ ssh_configs_dir }}/ssh_config"
  roles:
    - russStarr.ssh_config_from_inventory
```

Results
-------
Before you probably had to specify many parameters to SSH into a server. For example:
```
ssh -i ~/.ssh/my_secret_key.pem cloud-user@my_server.domain.tld
```

Assuming you have the Role Consumed Variables defined in your Ansible inventory file, it would simply your SSH command to:
```
ssh my_server
```

License
-------

Apache 2.0

Author Information
------------------

N/A
