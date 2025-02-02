---
title: Rclone S3 Mounts
description: This script configures Rclone to provide several S3-compliant mounts by leveraging CloudFlare R2
sidebar_label: 50 Rclone S3 Mounts
slug: /scripts/after/run_onchange_after_50-rclone.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_50-rclone.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_50-rclone.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_50-rclone.sh.tmpl
---
# Rclone S3 Mounts

This script configures Rclone to provide several S3-compliant mounts by leveraging CloudFlare R2

## Overview

Install Doctor leverages Rclone and CloudFlare R2 to provide S3-compliant bucket mounts that allow you to retain stateful files and configurations.
In general, these buckets are used for backing up files like your browser profiles, Docker backup files, and other files that cannot be stored as
as code in your Install Doctor fork.

This script sets up Rclone to provide several folders that are synchronized with S3-compliant buckets (using CloudFlare R2 by default).
The script ensures required directories are created and that proper permissions are applied. This script will only run if `rclone` is
available in the `PATH`. It also requires the user to provide `CLOUDFLARE_R2_ID` and `CLOUDFLARE_R2_SECRET` as either environment variables
or through the encrypted repository-fork-housed method detailed in the [Secrets documentation](https://install.doctor/docs/customization/secrets).

## Mounts

The script will setup five mounts by default and enable / start `systemd` services on Linux systems so that the mounts are available
whenever the device is turned on. The mounts are:

| Mount Location        | Description                                                                                                           |
|-----------------------|-----------------------------------------------------------------------------------------------------------------------|
| `/mnt/s3-private`     | Private system-wide bucket used for any private files that should not be able to be accessed publicly over HTTPS      |
| `/mnt/s3-public`      | Public system-wide bucket that can be accessed by anyone over HTTPS with the bucket's URL (provided by CloudFlare R2) |
| `/mnt/s3-docker`      | Private system-wide bucket used for storing Docker-related backups / files                                            |
| `/mnt/s3-system`      | Private system-wide bucket similar to `/mnt/s3-private` but intended for system file backups                          |
| `$HOME/.local/mnt/s3` | Private user-specific bucket (used for backing up application settings)                                               |

## Permissions

The system files are all assigned proper permissions and are owned by the user `rclone` with the group `rclone`. The exception to this is the
user-specific mount which uses the user's user name and user group.

## Samba

If Samba is installed, then by default Samba will create two shares that are symlinked to the `/mnt/s3-private` and `/mnt/s3-public`
buckets. This feature allows you to easily access the two buckets from other devices in your local network. If Rclone buckets are not
available then the Samba setup script will just create regular empty folders as shares.

## Notes

* The mount services all leverage the executable found at `$HOME/.local/bin/rclone-mount` to mount the shares.

## Links

* [Rclone mount script](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_local/bin/executable_rclone-mount)
* [Rclone default configurations](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/rclone)
* [Rclone documentation](https://rclone.org/docs/)



## Source Code

```
{{- if ne .host.distro.family "windows" -}}
#!/usr/bin/env bash
# @file Rclone S3 Mounts
# @brief This script configures Rclone to provide several S3-compliant mounts by leveraging CloudFlare R2
# @description
#     Install Doctor leverages Rclone and CloudFlare R2 to provide S3-compliant bucket mounts that allow you to retain stateful files and configurations.
#     In general, these buckets are used for backing up files like your browser profiles, Docker backup files, and other files that cannot be stored as
#     as code in your Install Doctor fork.
#
#     This script sets up Rclone to provide several folders that are synchronized with S3-compliant buckets (using CloudFlare R2 by default).
#     The script ensures required directories are created and that proper permissions are applied. This script will only run if `rclone` is
#     available in the `PATH`. It also requires the user to provide `CLOUDFLARE_R2_ID` and `CLOUDFLARE_R2_SECRET` as either environment variables
#     or through the encrypted repository-fork-housed method detailed in the [Secrets documentation](https://install.doctor/docs/customization/secrets).
#
#     ## Mounts
#
#     The script will setup five mounts by default and enable / start `systemd` services on Linux systems so that the mounts are available
#     whenever the device is turned on. The mounts are:
#
#     | Mount Location        | Description                                                                                                           |
#     |-----------------------|-----------------------------------------------------------------------------------------------------------------------|
#     | `/mnt/s3-private`     | Private system-wide bucket used for any private files that should not be able to be accessed publicly over HTTPS      |
#     | `/mnt/s3-public`      | Public system-wide bucket that can be accessed by anyone over HTTPS with the bucket's URL (provided by CloudFlare R2) |
#     | `/mnt/s3-docker`      | Private system-wide bucket used for storing Docker-related backups / files                                            |
#     | `/mnt/s3-system`      | Private system-wide bucket similar to `/mnt/s3-private` but intended for system file backups                          |
#     | `$HOME/.local/mnt/s3` | Private user-specific bucket (used for backing up application settings)                                               |
#
#     ## Permissions
#
#     The system files are all assigned proper permissions and are owned by the user `rclone` with the group `rclone`. The exception to this is the
#     user-specific mount which uses the user's user name and user group.
#
#     ## Samba
#
#     If Samba is installed, then by default Samba will create two shares that are symlinked to the `/mnt/s3-private` and `/mnt/s3-public`
#     buckets. This feature allows you to easily access the two buckets from other devices in your local network. If Rclone buckets are not
#     available then the Samba setup script will just create regular empty folders as shares.
#
#     ## Notes
#
#     * The mount services all leverage the executable found at `$HOME/.local/bin/rclone-mount` to mount the shares.
#
#     ## Links
#
#     * [Rclone mount script](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_local/bin/executable_rclone-mount)
#     * [Rclone default configurations](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/rclone)
#     * [Rclone documentation](https://rclone.org/docs/)

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

if command -v rclone > /dev/null; then
    logg info 'Ensuring /var/cache/rclone exists'
    sudo mkdir -p /var/cache/rclone
    sudo chmod 700 /var/cache/rclone
    sudo chown -Rf rclone:rclone /var/cache/rclone

    logg info 'Ensuring /var/log/rclone exists'
    sudo mkdir -p /var/log/rclone
    sudo chmod 700 /var/log/rclone
    sudo chown -Rf rclone:rclone /var/log/rclone

    logg info 'Adding ~/.local/bin/rclone-mount to /usr/local/bin'
    sudo cp -f "$HOME/.local/bin/rclone-mount" /usr/local/bin/rclone-mount
    sudo chmod +x /usr/local/bin/rclone-mount

    logg info 'Adding ~/.config/rclone/rcloneignore to /etc/rcloneignore'
    sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/rclone/rcloneignore" /etc/rcloneignore
    sudo chmod 644 /etc/rcloneignore

    logg info 'Adding ~/.config/rclone/system-rclone.conf to /etc/rclone.conf'
    sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/rclone/system-rclone.conf" /etc/rclone.conf

    ### Add / configure service files
    if [ -d /etc/systemd/system ]; then
        find "${XDG_CONFIG_HOME:-$HOME/.config}/rclone/system" -mindepth 1 -maxdepth 1 -type f | while read RCLONE_SERVICE; do
            ### Add systemd service file
            logg info "Adding S3 system mount service defined at $RCLONE_SERVICE"
            FILENAME="$(basename "$RCLONE_SERVICE")"
            SERVICE_ID="$(echo "$FILENAME" | sed 's/.service//')"
            sudo cp -f "$RCLONE_SERVICE" "/etc/systemd/system/$(basename "$RCLONE_SERVICE")"

            ### Ensure mount folder is created
            logg info "Ensuring /mnt/$SERVICE_ID is created with proper permissions"
            sudo mkdir -p "/mnt/$SERVICE_ID"
            sudo chmod 770 "/mnt/$SERVICE_ID"
            sudo chown -Rf rclone:rclone "/mnt/$SERVICE_ID"

            ### Enable / restart the service
            logg info "Enabling / restarting the $SERVICE_ID S3 service"
            sudo systemctl enable "$SERVICE_ID"
            sudo systemctl restart "$SERVICE_ID"
        done

        ### Add user Rclone mount
        logg info 'Adding user S3 rclone mount (available at ~/.local/mnt/s3)'
        sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/rclone/s3-user.service" "/etc/systemd/system/s3-${USER}.service"
        logg info 'Enabling / restarting the S3 user mount'
        sudo systemctl enable "s3-${USER}"
        sudo systemctl restart "s3-${USER}"
    fi
else
    logg info '`rclone` is not available'
fi

{{ end -}}
```
