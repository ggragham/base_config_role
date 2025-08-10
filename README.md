Base Config Role
=========

Basic Linux system configuration.

Overview
--------

**Authentication** – Enables passwordless sudo for specified users or groups.  
**Package Management** – Installs and removes packages for Debian-based and RHEL-based systems, updates the system.  
**WireGuard** – Deploys WireGuard VPN configuration file and enables the service.  
**OpenVPN** – Deploys OpenVPN client configuration file and enables the service.  
**Storage** – Unlocks and mounts LUKS devices, generic filesystems, and Btrfs subvolumes.  
**SFTP** – Creates shared folders for SFTP access.  
**SAMBA** – Creates shared folders over SMB/CIFS (WIP).  

Role Variables
--------------
```yml
#################################
###  Auth Configuration Vars  ###
#################################
AUTH_CONFIGURATION_ENABLED: false  # Enable or disable auth configuration.

#################################
###  Package Management Vars  ###
#################################
PKGS_MANAGE_ENABLED: false  # Enable or disable PKGs management tasks.

PKGS_DEBIAN_REMOVAL_LIST:  # Packages to remove on Debian.
  - nano
PKGS_DEBIAN_INSTALLATION_LIST:  # Packages to install on Debian.
  - bash-completion
  - curl
  # - duf
  - vim

PKGS_REDHAT_REMOVAL_LIST:  # Packages to remove on RHEL-based system
  - nano
PKGS_REDHAT_INSTALLATION_LIST:  # Packages to install on RHEL-based system.
  - bash-completion
  - curl
  - vim

###############################
###  WireGuard Config Vars  ###
###############################
WIREGUARD_ENABLED: false  # Enable or disable WireGuard VPN.
WIREGUARD_CONFIG_PATH_SRC: /path/to/config/wg0.conf  # Source path for WireGuard configuration.

#############################
###  OpenVPN Config Vars  ###
#############################
OPENVPN_ENABLED: false  # Enable or disable OpenVPN VPN.
OPENVPN_CONFIG_PATH_SRC: /path/to/config/client0.conf  # Source path for OpenVPN configuration.

#############################
###  Storage Config Vars  ###
#############################
STORAGE_ENABLED: false  # Enables or disables external storages mounting.

## If no LUKS-encrypted storages are used, leave this list empty:
# STORAGE_LUKS_LIST: []
STORAGE_LUKS_LIST:  # List of LUKS encrypted block devices.
  - label: crypt_data01  # Label for the LUKS device.
    keyfile_src: /path/to/keys/data01.key  # Path to the keyfile source.
    keyfile_dest: /root/data01.key  # Path to store the keyfile on the target machine.
    mapper: mapper_disk01  # Device mapper name for the LUKS device.

  - label: crypt_data02
    keyfile_src: /path/to/keys/data02.key
    keyfile_dest: /root/data02.key
    mapper: mapper_disk02

## If no storages with generic FS (like ext4, xfs, fat) are used, leave this list empty:
# STORAGE_GENERIC_FS_LIST: []
STORAGE_GENERIC_FS_LIST:  # List of generic filesystem storage configurations.
  - label: data01  # Label for the filesystem.
    owner: root  # Owner of the mounted filesystem.
    mount_point: /mnt/data01  # Mount point for the filesystem.
    fstype: ext4  # Filesystem type.
    opts: defaults  # Mount options.

  - label: data03
    owner: root
    mount_point: /mnt/data03
    fstype: ext4
    opts: defaults

## If no storages with BTRFS subvols are used, leave this list empty:
# STORAGE_BTRFS_LIST: []
STORAGE_BTRFS_LIST:  # List of BTRFS subvolume configurations.
  - label: data02  # Label for the BTRFS subvolume.
    owner: root  # Owner of the subvolume.
    subvol: '@cache'  # Name of the BTRFS subvolume.
    mount_point: /mnt/subvol2/cache  # Mount point for the subvolume.
    compression: zstd:1  # Compression method used.

  - label: data02
    owner: root
    subvol: '@db'
    mount_point: /mnt/subvol2/db
    compression: zstd:1

  - label: data04
    owner: root
    subvol: '@media'
    mount_point: /mnt/subvol4/media
    compression: zstd:1

  - label: data04
    owner: root
    subvol: '@backup'
    mount_point: /mnt/subvol4/backup
    compression: zstd:1

##########################
###  SFTP Config Vars  ###
##########################
SFTP_ENABLED: false  # Enable or Disable SFTP sharing.

## If no SFTP groups are used, leave this list empty:
# SFTP_GROUPS: []
SFTP_GROUPS:  # List of SFTP groups.
  - name: sftpgroup  # Name of the SFTP group.

## If no SFTP users are used, leave this list empty:
# SFTP_USERS: []
SFTP_USERS:  # List of SFTP users.
  - name: sftpuser  # Name of the SFTP user.
    password: foobar123  # Password for the user (plain text; will be hashed).
    group: sftpgroup  # Group to which the user belongs.

  - name: sftpuser2
    password: foobar321

SFTP_SHARES_LIST:  # List of SFTP user or group configurations.
  - type: user  # Type of match (user/group).
    name: sftpuser  # Name of the user or group.
    chroot: /mnt/sftp_chroot  # Chroot directory for the SFTP share.
    subdir: share  # User's working directory inside chroot.
    mode: '0750'  # Permissions mode for the chroot directory.
    password_auth_enabled: true  # Enable or disable password authentication.

  - type: user
    name: sftpuser2
    chroot: /mnt/sftp_chroot
    subdir: data
    mode: '0750'
    password_auth_enabled: true

  - type: group
    name: sftpgroup
    chroot: /mnt/sftp_share
    subdir: media
    mode: '0570'
    password_auth_enabled: false

#################################
###  SAMBA Config Vars (WIP)  ###
#################################
SAMBA_ENABLED: false  # Enable or disable SAMBA sharing.
SAMBA_WORKGROUP: WORKGROUP  # Workgroup name for the SAMBA server.
SAMBA_SERVER_STRING: SAMBA Share  # Description string for the SAMBA server.

## If no SAMBA groups are used, leave this list empty:
# SAMBA_GROUPS: []
SAMBA_GROUPS:  # List of SAMBA groups.
  - name: smbgroup  # Name of the SAMBA group.

## If no SAMBA users are used, leave this list empty:
# SAMBA_USERS: []
SAMBA_USERS:  # List of SAMBA users.
  - name: smbuser  # Name of the SAMBA user.
    password: foobar123  # Password for the user.
    groups: smbgroup  # Groups to which the user belongs.

  - name: user2
    password: foobar321

SAMBA_SHARES_LIST:  # List of SAMBA shares.
  - name: data  # Name of the SAMBA share.
    path: /mnt/smb_data  # Path to the shared directory.
    allow_guests: false  # Allow guest access.
    browseable: false  # Make the share browseable.
    writable: false  # Allow writing to the share.
    valid_users: smbuser @smbgroup  # Valid users for the share.
    force_user: smbuser  # Force file creation as this user.
    force_group: smbgroup  # Force file creation as this group.
    create_mask: '0660'  # Permissions mask for newly created files.
    directory_mask: '0770'  # Permissions mask for new directories.

  - name: share
    path: /mnt/smb_share
    allow_guests: true
    browseable: true
    writable: true
    force_user: nobody
```

Installing Role
---------------

```bash
ansible-galaxy install git+https://github.com/ggragham/base_config_role.git
```

Example Playbook
----------------

```yml
  - hosts: servers
    roles:
       - role: base_config_role
```

License
-------

GPL

Author Information
------------------

[Grell Gragham](https://github.com/ggragham)
