# Playground for Azure ARM templates

## Template for cloud-init based VM

Create VM with one data disc and automatic configuration done via cloud-init. VM deployment expects:
* `location` - resource location
* `vm` - name of VM
* `vmSize` - VM size
* `zone` - Availability zone
* `username` - username for linuz server
* `sshkey` - publik ssh key for username
* `nsg` - existing NSG for VM - we will need ResourceId of existing NSG
* `subnet` - existing VNET + Subnet - ResourceID of existing Subnet where template will place network adapter
* `logWorkspace` - existing Log Analytics workspace - we will need ResourceId of existing workspace
* `cloudinitbase64` - base64 encoded cloud-init file, see example content of cloud-init there:

**cloud-init**
```
#cloud-config

write_files:
    - content: |
          username=#########
          password=#########
      path: /etc/smbcredentials/applog.cred

disk_setup:
    /dev/disk/azure/scsi1/lun0:
        table_type: gpt
        layout: True
        overwrite: True
fs_setup:
    - device: /dev/disk/azure/scsi1/lun0
      partition: 1
      filesystem: ext4
mounts:
    - ["/dev/disk/azure/scsi1/lun0-part1", "/var/www", auto, "defaults,noexec,nofail"]
    - ["//#########.file.core.windows.net/logs", "/storage", cifs, "nofail,vers=3.0,credentials=/etc/smbcredentials/applog.cred,dir_mode=0777,file_mode=0777,serverino"]
    - ["//##########.file.core.windows.net/appfiles", "/var/lib/php/sessions", cifs, "nofail,vers=3.0,credentials=/etc/smbcredentials/applog.cred,dir_mode=0777,file_mode=0777,serverino"]

packages:
    - apache2
    - php7.2
    - git
```

[![Deploy to Azure](https://azuredeploy.net/deploybutton.png)](https://azuredeploy.net/?repository=https%3A%2F%2Fraw.githubusercontent.com%2Fvalda-z%2Fazure-infra-payground%2Fmaster%2Fcloud-init-vm%2Fazuredeploy.json)