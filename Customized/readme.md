# Instructions

## Environment setup

### Inventories:

Add network OS information to the inventory. Below is a sample:

```yaml
---
ansible_network_os: nxos
ansible_connection: network_cli
```

Add hostname information under the host definition. Below is a sample:

```yaml
---
ansible_network_hostname: sample_hostname
```
### SFTP setup

Create a customized Credential Type using the template file SFTP_Cred_type.yml.
SFTP-related variables will be passed through the SFTP credential.

The SFTP path should be provided via Extra Variables. Below is a sample:

```yaml
sftp_path: ansible/backup
```


## show.yml

The commands executed by Ansible are provided via Extra Variables. Below is a sample:

```yaml
---
cli_cmd:
  - show int description | inc down
  - show inventory
```
## config.yml

The configuration commands executed by Ansible are provided via Extra Variables. Below is a sample.

**!!!⚠️Please make sure there is a space before "description"**
```yaml
---
config_cmds:
  - |
     interface Gi1/1/1
      description test1
  - |
     interface Gi1/1/2
      description test2
```
