# Golden Images
A key goal for this homelab is immutable infrastructure - servers are never
modified directly once they are deployed. Instead, all configuration changes are
implemented on golden images that are subsequently provisioned on all systems.
This prevents configuration drift between systems and allows for versioned
changes that are easy to rollback.

On Proxmox, this involves the creation of VM and LXC golden images.

The image building process is quick, automated and well tested in a CI/CD
pipeline:
1. Add new configuration to playbook
2. Build the golden image and store it in an artifact repository
3. Perform automated tests for new changes
4. Backup all persistent data
5. Destroy existing servers and re-create with new images

## Roles
There exist three roles for building a golden image
1. create_lxc - Create and start temporary LXC
2. template_lxc - Stop and template temporary LXC
3. bootstrap - Bootstrap temporary host with configuration

## LXC

Ansible is used to build LXC templates.

A temporary LXC container is created and bootstrapped. After configuratiuon, it
is stopped and Proxmox create a template with `vzdump`, before deleting the
container.

### Usage
1. Run `make install` to install all Ansible roles and collections.
2. Input [variables](#variables) in `images/vars.yml`
3. Run `make image`. This will prompt for your Proxmox sudo password (if not
   root@pam).

### Notes
When testing or debugging, you might destroy and recreate a new LXC with the
same IP in a short span of time. If the temp container is unreachable, ensure
that your router's ARP table cache is cleared.

It is advisable to create a non-root user in Proxmox (eg. `ansible@pam`) and use
an API token to authenticate with Proxmox. This are input in
`inventory/hosts.yml`.

### Variables
#### Authentication
| Variable                 | Type   | Description                |
| ------------------------ | ------ | -------------------------- |
| proxmox_api_host         | string | Target host of PVE cluster |
| proxmox_api_user         | string | User to authenticate with (eg. root@pam)  |
| proxmox_api_password     | string | Password                   |
| proxmox_api_token_id     | string | API token ID               |
| proxmox_api_token_secret | string | API token secret           |

#### Proxmox Parameters
| Variable           | Type   | Description                  |
| ------------------ | ------ | ---------------------------- |
| proxmox_node       | string | PVE node to operate on       |
| proxmox_storage    | string | Target storage               |

#### Temp Template Parameters
| Variable        | Type   | Description                                          |
| --------------- | ------ | ---------------------------------------------------- |
| base_image      | string | Existing base image to build on                      |
| temp_vmid       | int    | Instance ID of temp LXC                              |
| temp_hostname   | string | Hostname of temp LXC                                 |
| temp_disk_size  | string | Hard disk size of format `<Storage>:<Size>`          |
| temp_ip_address | string | IPv4 address for temp LXC of format `cidr/subnet_mask` |
| temp_gateway    | string | Gateway address for temp LXC                         |

#### Template Storage Parameters
| Variable          | Type   | Description                                |
| ----------------- | ------ | ------------------------------------------ |
| dump_dir          | string | Directory to store template                |
| template_name     | string | Name of built template. Suffixed with date |

#### Bootstrap Configuration
| Variable             | Type   | Description                                    |
| -------------------- | ------ | ---------------------------------------------- |
| template_user        | string | Username of new user                           |
| template_password    | string | Temporary password of new user                 |
| template_uid         | int    | UID of new user                                |
| apps                 | list   | Packages to install                            |
| force_reset_password | bool   | Prompt user to reset password on first login   |
| git_email            | string | Git email config                               |
| git_user             | string | Git username config                            |
| ssh_key_file         | list(string) | Absolute paths to SSH public key files that are to be added to template_user's authorized_keys file|

>The `bootstrap` role also allows the direct addition of SSH public key files in
>`bootstrap/files/` instead of passing a file path.

## VM
TODO
