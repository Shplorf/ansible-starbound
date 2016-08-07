Starbound Dedicated Server
==========================

Role for configuring a Starbound dedicated server and optionally backing the universe up to S3

Supported platforms: Ubuntu 14.04.5 LTS (Trusty Tahr)

[![Build Status](https://travis-ci.org/Shplorf/ansible-starbound.svg?branch=master)](https://travis-ci.org/Shplorf/ansible-starbound)

Requirements
------------
Ansible should be configured to [allow pipelining](https://docs.ansible.com/ansible/intro_configuration.html#pipelining).

Steam username and password should be defined (see variables section below).
For this role to work, the Steam account cannot have Steam Guard on (otherwise steamcmd will prompt for an access code that is emailed to you, no easy way to automate that). Because of this I recommed having a dedicated Steam account that only owns Starbound, and using that account's credentials. I repeat, I DO NOT RECOMMEND TURNING STEAM GUARD OFF ON YOUR MAIN ACCOUNT.

If using the S3 backup/restore functionality, you need to have a bucket created with a name that matches the starbound_backup_s3_bucket variable, and AWS key credentials defined (see variables section below).

Role Variables
--------------
- starbound_server_config_file: Absolute or relative path to the local [server configuration](http://starbound.gamepedia.com/Starbound.config) file that will be copied over to your server.
- starbound_backup_s3_bucket: (Optional) Name of the S3 bucket you wish to backup your universe to

- starbound_update: (Defaults to false) If set to true, will check for updates. Otherwise, if Starbound is already installed it will not be updated.
- starbound_backup: (Defaults to false) If set to true, the current universe will be backed up to S3
- starbound_restore: (Defaults to false) If set to true, the most recent version of the universe backup will be restored from S3

It is recommended that the following variables are stored in an encrypted vault YAML file:
- starbound_steam_user: Steam username of a user that owns Starbound and has Steam Guard turned off (see requirements section for explanation)
- starbound_steam_pass: Password of the above user
- starbound_aws_access_key: (Optional) AWS access key of a user with access to the S3 bucket specified in the starbound_backup_s3_bucket variable
- starbound_aws_secret_key: (Optional) AWS secret key of the above user

Example Playbooks
-----------------
Without S3 backup:
```
- name: Apply starbound configuration
  hosts: server
  user: user
  become: yes
  become_method: sudo
  pre_tasks:
    - name: Include Steam and AWS credentials as variables
      include_vars: credentials.yml
  roles:
    - { 
      role: Shplorf.starbound,
      starbound_server_config_file: "files/starbound_server.config"
    }
```
With S3 backup:
```
- name: Apply starbound configuration
  hosts: server
  user: user
  become: yes
  become_method: sudo
  pre_tasks:
    - name: Include Steam and AWS credentials as variables
      include_vars: credentials.yml
  roles:
    - { 
      role: Shplorf.starbound,
      starbound_backup_s3_bucket: "shplorf-starbound",
      starbound_server_config_file: "files/starbound_server.config"
    }
```
With EC2 dynamic inventory and S3 backup:
```
- name: Apply starbound configuration
  hosts: tag_Role_starbound
  user: ubuntu
  become: yes
  become_method: sudo
  pre_tasks:
    - name: Include Steam and AWS credentials as variables
      include_vars: credentials.yml
  roles:
    - { 
      role: Shplorf.starbound,
      starbound_backup_s3_bucket: "shplorf-starbound",
      starbound_server_config_file: "files/starbound_server.config"
    }
```

Example Commands
----------------
Execute one of the above playbooks with the goal of ensuring a Starbound server is up and running:

```ansible-playbook main.yml```

Execute one of the above playbooks with the goal of ensuring a Starbound server is up and running AND running the latest version of the dedicated server:

```ansible-playbook main.yml --extra-vars "starbound_update=true"```

Execute one of the above playbooks with the goal of ensuring a Starbound server is up and running AND back up the current universe to S3:

```ansible-playbook main.yml --extra-vars "starbound_backup=true"```

Execute one of the above playbooks with the goal of ensuring a Starbound server is up and running AND restore the most recent universe version from S3:

```ansible-playbook main.yml --extra-vars "starbound_restore=true"```

License
-------
MIT
