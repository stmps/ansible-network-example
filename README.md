# ansible-vyos-example

This repository contains and Ansible Tower Playbook to demonstrate the capability of managing and configuring networking devices. 

![Ansible](docs/images/network.png)

## Overview

* make a global change - update the banner accross all routers
* deploy a new site - configure routers and setup routing
* backup configuration and diff against gold configs
* remediate configuration from gold config
* generate a network diagram based on LLDP information

## Pre-reqs

This demonstration can be easily deployed using Vagrant with Virtualbox. The Ansible Tower install is outside the scope of this deployment and must have access to the vboxnet1 hostonly interfaces of the routers. Ansible Tower also requires postfix to be configured as an email relay, to allow Tower Playbooks to send emails.

Vagrant assumes that virtualbox has the following networks available:
* vboxnet1 (hostonly)
* vboxnet2 (hostonly)
* vboxnet3 (hostonly)
* intnet (internal only)


```bash
for i in {1..4}; do vboxmanage hostonlyif create; done

vboxmanage hostonlyif ipconfig vboxnet1 --ip 192.168.128.0 --netmask 255.255.255.0
vboxmanage hostonlyif ipconfig vboxnet2 --ip 192.168.129.0 --netmask 255.255.255.0
vboxmanage hostonlyif ipconfig vboxnet3 --ip 192.168.130.0 --netmask 255.255.255.0
vboxmanage hostonlyif ipconfig vboxnet4 --ip 192.168.131.0 --netmask 255.255.255.0
```

## Deploy

```bash
cd vagrant/tower
vagrant up
...
...
    tower-vagrant: Finished
    tower-vagrant:
    tower-vagrant:   Welcome to Ansible Tower!
    tower-vagrant:
    tower-vagrant:   Log into the web interface here:
    tower-vagrant:
    tower-vagrant:     https://10.42.0.42/
    tower-vagrant:
    tower-vagrant:     Username: admin
    tower-vagrant:     Password: <random string>
    tower-vagrant:
    tower-vagrant:   The documentation for Ansible Tower is available here:
    tower-vagrant:
    tower-vagrant:     http://www.ansible.com/tower/
    tower-vagrant:
    tower-vagrant:   For help, visit  http://support.ansible.com/


cd ../..

ansible-playbook -i inventory configure-tower.yml
PLAY [install updated banner] ***********************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************
ok: [tower]

TASK [tower_config : get tower password] ***********************************************************************************************************************************************************
changed: [tower]

TASK [tower_config : Create tower organization] ***********************************************************************************************************************************************************
changed: [tower]

TASK [tower_config : Add tower project] ***********************************************************************************************************************************************************
changed: [tower]

TASK [tower_config : Create a valid vyos-ssh credential] ***********************************************************************************************************************************************************
changed: [tower]

PLAY RECAP ************************************************************************************************************************************************
tower                      : ok=6    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```bash
cd vagrant/routers
vagrant up

cd ../linux_machines
vagrant up
```

## Ansible Tower

*vagrant*
Vagrant will provision a Tower instance that you can access via: *https://127.0.0.1:8443*
Note - It will take a few minutes for Tower to become available. The username and password will be displayed once vagrant has finished provisioning.

You will need to add your license file, create the credentials and import the playbook (these steps will be automated.)

*Credentials*
* git-ssh | Machine | add the ssh key to pull and update your private repo to host the router configurations
* ssh | Network | set username: vyos password: vyos 

*Inventory*
- Group: Routers
- Hosts:
  - R1 - 192.168.128.1
  - R2 - 192.168.128.2

*vars for hosts - e.g. R1*
```yaml
---
ansible_host: 192.168.128.1
ansible_network_os: vyos
ansible_user: vyos
config_git_repo: "set the git repo to store and pull router configurations"
```

*git vars*
If you want to runt he play that can compare and diff router configuraitons, you need a place to upload and store the playbooks. Create a repo in github and set the config_git_repo variable. Make sure the git-ssh key listed above (Credentials) is associated with the git_repo.
```yaml
---
config_git_repo: "set the git repo to store and pull router configurations"
```

#### Guide to configure Tower to send email

The link below explains how to configure postfix on the local tower node to send emails via GMail.
https://digitalave.github.io/spring/2019/10/25/Setup-Postfix-To-Send-Emails-Using-Gmail-Relay.html

## Connecting to the Trusted Client 

```bash
vagrant ssh client -- -Y firefox http://server.rh.com --no-remote --no-xshm
```

#### Trusted Connection

The Trusted Client Connection should always be successful.

![Image of Client Connection](docs/images/client.png)


## Connecting to the Untrusted Client 

```bash
vagrant ssh bad -- -Y firefox http://server.rh.com --no-remote --no-xshm
```

#### Untrusted Connection

The Untrusted Client Connection should only work when the remove-firewall playbook has been run.

![Image of Untrusted Connection](docs/images/untrusted.png)

## Networking Diagram

![Image of Network Scenario](docs/images/scenario.png)

## Playbooks

### vars

Set the notification email address within the vault.yml file
*vault.yml*
```bash
VAULT_EMAIL_ADDRESS: "email address"
```

*groups_vars/all.yml*
```bash
config_git_repo: "set the git repo to store and pull router configurations"
```

### global-change.yml

![Image of Global Change](docs/images/global.png)

### provision-site.yml

![Image of Site Provision](docs/images/new_site.png)

### remove-firewall.yml

![Remove the firewall config](docs/images/firewall.png)

### backup-config-diff.yml

#### Backup

![Backup config and diff to gold](docs/images/backup_config.png)

#### Diff Notification

![Backup config and diff to gold](docs/images/email.png)

### restore-config.yml

![Restore config from gold config](docs/images/remediate.png)

### generate LLDP Network Diagram 

![Generate Network Diagram](docs/images/gen_network_diagram.png)

