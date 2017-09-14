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

Upgrading
---------
Using one of the Installation commands previously listed, just add `--force` to your command.

Role Variables
--------------

### User Defined Variables

|Variable|Default|Description|
|---|---|---|
| `ssh_configs_dir` | `{{playbook_dir}}/ssh_configs` | Location where the group specific SSH config files are stored prior to merging to a single SSH config file. |
|`ssh_config_file`|`{{ssh_configs_dir}}/ssh_config`|Where should the SSH client configuration be written to? Most implementations use `~/.ssh/config` so you can change this if you want.|
|`inventory_groups`|`["all"]`|Which inventory groups should we read to create SSH client configuration for? By default the built-in group `all` will be used since it should always be valid. `ungrouped` is also a built-in group name. To get a full list of groups in your Ansible directory, use `ansible -m debug -a 'var=groups.keys()\|sort' localhost`.|
|`keepgroupnames`|`"False"`|Should ansible groupname(s) be used to create a pattern for hostname ? When a server is part of several groups, then additionnal patterns will be created to match each and every group. When set to `"True"` and with inventory file example below, `ssh server1` **and** `ssh production.server1` will both work.|


### Role Consumed Variables
|Variable|Usage|Description|
|---|---|---|
|`inventory_hostname_short`|Required but usually set automatically.|This name will be used to define the `Host` section of the SSH client configuration.|
|`ansible_host` (preferred)<br> `ansible_ssh_host`|optional|If set, a `HostName` value will be set for the host. This is useful when you do not have the host in DNS.|
|`ansible_user` (preferred)<br>`ansible_ssh_user`|optional|If set, a `User` value will be set for the host.|
|`ansible_port` (preferred)<br>`ansible_ssh_port`|optional|If set, a `Port` value will be set for the host.|

All Role Consumed Variables can be defined in your [Ansible inventory file](http://docs.ansible.com/ansible/latest/intro_inventory.html).

Here is an example of an Ansible inventory file making use of all the variables.
```
localhost ansible_connection=local

[production]
server1.domain.tld ansible_host=10.10.10.10
server2.domain.tld
server3.domain.tld

[production:vars]
ansible_ssh_private_key_file=~/.ssh/production_key.pem

[staging]
server4.domain.tld ansible_ssh_user=mfischer
server5.domain.tld ansible_ssh_port=1998
server6.domain.tld ansible_ssh_host=127.0.0.1

[staging:vars]
ansible_ssh_private_key_file=~/.ssh/staging_key.pem
ansible_port=2222

[all:vars]
ansible_user=ansible
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

1.  First create the folder to store local SSH client configuration files by group.
2.  Go through all groups defined in `inventory_groups` and create a local SSH client configuration.
3.  Combine all the SSH client configuration files and merge them into the User Defined `ssh_config_file`, which can be ~/.ssh/config for convenience.

> Note: The `blockinfile` module will manage blocks of configuration in the `ssh_config_file`. You can safely have existing configuration in your `ssh_config_file` because `blockinfile` will only look for config blocks as defined be the `{mark}` value.

Example Playbook
----------------

Here are two example plays that you may wish to add to one of your existing playbooks. All the variables are defined for illustration purposes. The role is intended to be used by localhost as the current user, though you can change that to suit your needs.

**Purpose:** Read inventory groups `production` and `staging` then write an SSH configuration to /tmp/ssh_configs/ssh_config.
```
---
- hosts: localhost
  become: "False"
  gather_facts: "False"
  vars:
    inventory_groups: ["production","staging"]
    keepgroupnames: "True"
    ssh_configs_dir: "/tmp/ssh_configs"
    ssh_config_file: "{{ ssh_configs_dir }}/ssh_config"
  roles:
    - russStarr.ssh_config_from_inventory
```

**Purpose:** Same as previous play except write the SSH configuration to the user's respective config at `~/.ssh/config`

```
---
- hosts: localhost
  become: "False"
  gather_facts: "False"
  vars:
    inventory_groups: ["production","staging"]
    keepgroupnames: "True"
    ssh_config_file: "~/.ssh/config"
  roles:
    - ssh_config_from_inventory
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
or
```
ssh anygroup.my_server
```

License
-------

Apache 2.0

Author Information
------------------

N/A
