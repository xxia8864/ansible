# Instructions

## Environment setup

### Inventories:

Add network OS information to inventory,below are the sample.

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


## show.yml
Please create a customized credential type using the template name "SFTP_Cred_type.yml" 

The command that ansbile run are in Extra Variables, below are the sample

```yaml
cli_cmd:
  - show int description | inc down
  - show inventory
```
