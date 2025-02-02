---
title: Fail2ban Configuration
description: Applies the system `fail2ban` jail configuration and then restarts the service
sidebar_label: 31 Fail2ban Configuration
slug: /scripts/after/run_onchange_after_31-fail2ban.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_31-fail2ban.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_31-fail2ban.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_31-fail2ban.sh.tmpl
---
# Fail2ban Configuration

Applies the system `fail2ban` jail configuration and then restarts the service

## Overview

Fail2ban is an SSH security program that temporarily bans IP addresses that could possibly be
attempting to gain unauthorized system access. This script applies the "jail" configuration
located at `home/private_dot_ssh/fail2ban/` to the system location. It then enables and restarts
the `fail2ban` configuration.

## Links

* [`fail2ban` configuration folder](https://github.com/megabyte-labs/install.doctor/tree/master/home/private_dot_ssh/fail2ban)



## Source Code

```
{{- if eq .host.distro.family "linux" -}}
#!/usr/bin/env bash
# @file Fail2ban Configuration
# @brief Applies the system `fail2ban` jail configuration and then restarts the service
# @description
#     Fail2ban is an SSH security program that temporarily bans IP addresses that could possibly be
#     attempting to gain unauthorized system access. This script applies the "jail" configuration
#     located at `home/private_dot_ssh/fail2ban/` to the system location. It then enables and restarts
#     the `fail2ban` configuration.
#
#     ## Links
#
#     * [`fail2ban` configuration folder](https://github.com/megabyte-labs/install.doctor/tree/master/home/private_dot_ssh/fail2ban)

# jail.local hash: {{- include (joinPath .host.home ".ssh" "fail2ban" "jail.local") | sha256sum -}}

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Restart fail2ban
function restartFail2Ban() {
    if [ -d /Applications ] && [ -d /System ]; then
        # macOS
        logg info 'Enabling the `fail2ban` Homebrew service'
        brew services start fail2ban
    else
        # Linux
        logg info 'Enabling the `fail2ban` service'
        sudo systemctl enable fail2ban
        logg info 'Restarting the `fail2ban` service'
        sudo systemctl restart fail2ban
    fi
}

### Update the jail.local file if environment is not WSL
if [[ ! "$(test -d /proc && grep Microsoft /proc/version > /dev/null)" ]]; then
    if [ -d /etc/fail2ban ]; then
        logg info 'Copying ~/.ssh/fail2ban/jail.local to /etc/fail2ban/jail.local'
        sudo cp -f "$HOME/.ssh/fail2ban/jail.local" /etc/fail2ban/jail.local
        restartFail2Ban
    elif [ -d /usr/local/etc/fail2ban ]; then
        logg info 'Copying ~/.ssh/fail2ban/jail.local to /usr/local/etc/fail2ban/jail.local'
        sudo cp -f "$HOME/.ssh/fail2ban/jail.local" /usr/local/etc/fail2ban/jail.local
        restartFail2Ban
    else
        logg warn 'Both the /etc/fail2ban (Linux) and the /usr/local/etc/fail2ban (macOS) folder do not exist'
    fi
else
    logg info 'Skipping sshd_config application since environment is WSL'
fi

{{ end -}}
```
