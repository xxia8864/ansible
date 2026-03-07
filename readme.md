# Ansible Network Automation Flow

This repository runs backup, show, config, and firmware workflows for IOS/NX-OS devices, then uploads artifacts to SFTP.

## Prerequisites

- Control node:
  - `ansible`
  - Python package: `scp` (`requirements.txt`)
- Inventory host vars:
```yaml
ansible_connection: network_cli
ansible_network_os: ios   # or nxos
ansible_network_hostname: sw01
```
- SFTP credentials passed by your platform (for example AWX/Tower credential type using `template/SFTP_Cred_Type.yml`):
  - `sftp_host`
  - `sftp_username`
  - `sftp_password`
  - `sftp_path` (also provided as extra var)

## Shared Runtime Flow

Most playbooks follow this pattern:

1. Generate timestamp (`tasks/t_time_stamp.yml`).
2. Create/add temporary SFTP host in-memory (`tasks/t_sftp.yml`, host: `sftp_target`).
3. Create files under `/tmp/tmpfiles`.
4. Upload to SFTP (`tasks/t_upload.yml`).
5. Delete local temporary files (`tasks/t_delete_file.yml`).

## Playbook Flows

## `show.yml`

Flow:

1. Build report header file on localhost: `/tmp/tmpfiles/show_output_<HHMMSS>.txt`.
2. Run each command in `cli_cmd` on each switch (`ios_command`/`nxos_command`).
3. Create per-host temporary files: `/tmp/tmpfiles/<report>-<hostname>.txt`.
4. Merge per-host files into final report file.
5. Upload final report to SFTP.
6. Delete final and per-host temp files in `post_tasks`.

Extra vars example:
```yaml
cli_cmd:
  - show int description | inc down
  - show inventory
```

## `backup.yml`

Flow:

1. Run `save_config.yml` first.
2. For each network device (`all:!temp`), run `cli_config` backup to `/tmp/tmpfiles/<hostname>_<HHMMSS>.txt`.
3. Upload backup file to SFTP date folder: `<sftp_path>/<YYYYMMDD>/`.
4. Delete local temp backup file in `post_tasks`.

## `config.yml`

Flow:

1. Run `backup.yml` first.
2. Push config snippets from `config_cmds` using `cli_config`.
3. Run `save_config.yml` after changes.

Extra vars example:
```yaml
config_cmds:
  - |
    interface Gi1/1/1
     description test1
  - |
    interface Gi1/1/2
     description test2
```

Note: keep indentation consistent in multiline config blocks.

## `firmware_upload.yml`

Flow:

1. Ensure `scp` is installed on localhost.
2. Optionally clean IOS flash space (`tasks/t_patch_clean_ios.yml`).
3. Download firmware from SFTP to controller `/tmp/tmpfiles/<firmware_filename>`.
4. Upload firmware to switches with `net_put` (throttled).
5. Delete local firmware temp file.
6. Verify MD5 on devices (`tasks/t_check_md5.yml`).

Required extra vars:
```yaml
firmware_filename: cat9k_iosxe.17.09.05.SPA.bin
md5: "<official_md5_value>"
sftp_path: ansible/firmware
```

## `firmware_patch_ios.yml`

Flow:

1. Run `backup.yml`.
2. Create/clear `/tmp/tmpfiles/patch_log.txt`.
3. Set startup-config behavior for IOS switches.
4. Upgrade even IOS group, then odd IOS group (`tasks/t_patch_ios.yml`).
5. In each patch task:
   - Capture pre-check interface status and upload it.
   - Delete temp pre-check file.
   - Verify firmware MD5.
   - Run install command.
   - Reset connection and run post-check confirmation.
6. Post cleanup (`tasks/t_patch_clean_ios.yml`).

## `firmware_patch_nxos.yml`

Flow:

1. Run `backup.yml`.
2. Create/clear `/tmp/tmpfiles/patch_log.txt`.
3. Upgrade even NX-OS group, then odd NX-OS group (`tasks/t_patch_nxos.yml`).
4. In each patch task:
   - Capture pre-check interface status (text + json) and upload text snapshot.
   - Delete temp pre-check file.
   - Verify firmware MD5.
   - Run `nxos_install_os`.
   - Reset connection and run post-check confirmation.
5. Optional post cleanup when `cleanup=true` (`tasks/t_patch_clean_nxos.yml`).

## Notes

- Date/time values are generated in `America/New_York`.
- Temporary files are always created under `/tmp/tmpfiles`.
- The `temp` inventory group is used for the runtime `sftp_target` host and is excluded from network device plays (`all:!temp`).
