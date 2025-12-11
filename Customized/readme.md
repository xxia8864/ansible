# Instructions

## Environment setup

### Inventories:

Add network OS information to inventory, below are the sample.

```yaml
---
ansible_network_os: nxos
ansible_connection: network_cli
```

Add hostname info under host, below are the sampe

```yaml
---
ansible_network_hostname: sample_hostname
```
### SFTP setup

Please create a customized **credential type** using the template named "SFTP_Cred_type.yml" 
The SFTP variables will be sent through SFTP credential. 

## show.yml

The command that ansbile run are in Extra Variables, below are the sample

```yaml
---
cli_cmd:
  - show int description | inc down
  - show inventory
```
## config.yml

The configuration that ansbile run are in Extra Variables, below are the sample

```yaml
---
config_cmds:
  - |
     interface Gi1/1/1
     ndescription test1
  - |
     interface Gi1/1/2
     ndescription test2
```
