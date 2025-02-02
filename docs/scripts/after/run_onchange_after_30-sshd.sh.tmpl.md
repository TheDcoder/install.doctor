---
title: SSHD Configuration
description: Applies SSHD system configuration and then restarts / enables the SSH server
sidebar_label: 30 SSHD Configuration
slug: /scripts/after/run_onchange_after_30-sshd.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_30-sshd.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_30-sshd.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_30-sshd.sh.tmpl
---
# SSHD Configuration

Applies SSHD system configuration and then restarts / enables the SSH server

## Overview

This script applies the SSH server MOTD banner and `sshd_config` (which are housed in the `home/private_dot_ssh/system` location)
to the system by copying the files to the system location and then restarting / enabling the system SSH server.

## Links

* [System SSHD configurations](https://github.com/megabyte-labs/install.doctor/tree/master/home/private_dot_ssh/system)



## Source Code

```
{{- if ne .host.distro.family "windows" -}}
#!/usr/bin/env bash
# @file SSHD Configuration
# @brief Applies SSHD system configuration and then restarts / enables the SSH server
# @description
#     This script applies the SSH server MOTD banner and `sshd_config` (which are housed in the `home/private_dot_ssh/system` location)
#     to the system by copying the files to the system location and then restarting / enabling the system SSH server.
#
#     ## Links
#
#     * [System SSHD configurations](https://github.com/megabyte-labs/install.doctor/tree/master/home/private_dot_ssh/system)

# sshd_config hash: {{- include (joinPath .host.home ".ssh" "system" "sshd_config") | sha256sum -}}
# banner hash: {{- include (joinPath .host.home ".ssh" "system" "banner") | sha256sum -}}

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Update /etc/ssh/sshd_config if environment is not WSL
if [[ ! "$(test -d /proc && grep Microsoft /proc/version > /dev/null)" ]]; then
    if [ -d /etc/ssh ]; then
        logg info 'Copying ~/.ssh/system/banner to /etc/ssh/banner'
        sudo cp -f "$HOME/.ssh/system/banner" /etc/ssh/banner

        logg info 'Copying ~/.ssh/system/sshd_config to /etc/ssh/sshd_config'
        sudo cp -f "$HOME/.ssh/system/sshd_config" /etc/ssh/sshd_config

        if command -v semanage > /dev/null; then
            logg info 'Apply SELinux configuration addressing custom SSH port'
            sudo semanage port -a -t ssh_port_t -p tcp {{ .host.ssh.port }}
            logg info 'Allow NIS SSHD'
            sudo setsebool -P nis_enabled 1
        fi

        ### Restart SSH server
        if [ -d /Applications ] && [ -d /System ]; then
            # macOS
            logg info 'Running `sudo launchctl stop com.openssh.sshd`'
            sudo launchctl stop com.openssh.sshd
            logg info 'Running `sudo launchctl start com.openssh.sshd`'
            sudo launchctl start com.openssh.sshd
        else
            # Linux
            logg info 'Enabling the `sshd` service'
            sudo systemctl enable sshd
            logg info 'Restarting the `sshd` service'
            sudo systemctl restart sshd
        fi
    else
        logg warn 'The /etc/ssh folder does not exist'
    fi
else
    logg info 'Skipping sshd_config application since environment is WSL'
fi

{{ end -}}
```
