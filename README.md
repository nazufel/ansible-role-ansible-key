# Ansible Role: Ansible Key

This Ansible Role is used to copy a public key to a remote host to be used by a pre-baked, interactive user that was created for administrative use by Ansible.

## Use Case

Ansible requires a user to be present on a remote host to connect via SSH. There is a global variable for this, `ansible_user`, to be used in the *inventory* file. However, sealing a Linux template machine in [oVirt](https://www.ovirt.org/documentation/vmm-guide/chap-Templates/) it removes the *.ssh* directories for all users and, thus, *authorized_keys* files from the designated, interactive user to be used by Ansible. This means that all users can only be accessed via password for ssh. I prefer to have a user with sudo permissions to be accessed via ssh key only. I always have an interactive user to be used by Ansible that can be accessed via public key without a password challange.

This role will connect to a newly deployed Linux server from a template where the *.ssh* was removed and use and a password to authenticate the connection to a pre-baked interactive user. Then it will create an *.ssh/authorized_keys* file with proper permissions and add the desired public ssh key to that user so that it can be accessed by other Ansible roles without needing a password.

## Requirements

* Interactive user on inventory target
* Password for above interactive user, stored in a password file, and (preferably) encrypted with Ansible [Vault](https://docs.ansible.com/ansible/2.4/vault.html)


## Role Variables

Variable Names | Required | Description
---------------| :------: | -----------
interactive_user | YES | The name of the interactive user to be used by Ansible and is the target of this Role.
interactive_password | YES | The password for the interactive user. This is needed to gain access to the interactive user so that this Role can add a public key to the user's `authorized_keys`.
interactive_public_key | YES | The public key to be placed into the interactive user's `authorized_keys` file.

## Dependencies

**None**

## Example Variable Files and Playbooks

### Example Variables Files (Pre Ansible-Vault Encryption of the Password File)

```yml
# vars/interactive_vars.yml
---
interactive_user: interactiveuser
interactive_public_key: "~/.ssh/interactive-key.pub"
```

```yml
# password.yml
---
interactive_pasword: alwaysencryptpasswordfile!
```

### Example Playbooks

#### Playbook with Separate Variables Files

Below is an example playbook using external variables files in the playbook, outside of the role.

```yml
- hosts: servers
  vars_files:
    - vars/interactive_vars.yml
    - password.yml
  roles:
    - { role: nazufel.ansible_role_ansible_key }
```

#### Playbook with Included Variables

Below is an example playbook including variables in the playbook file. All required variables in included, except for the `interactive_password` variable

```yml
# ansible-key.yml
- hosts: servers
  name: Add Ansible Key to Authorized Keys Playbook
  vars_files:
    - passwords.yml
  vars:
    - interactive_user: "ansible"
    - interactive_public_key: "~/.ssh/interactive_key.pub"
  roles:
    - { role: nazufel.ansible_role_ansible_key }
```

## License

MIT

## Author Information

This role was created in 2018 by Ryan Ross, aka [Nazufel](https://github.com/nazufel).
