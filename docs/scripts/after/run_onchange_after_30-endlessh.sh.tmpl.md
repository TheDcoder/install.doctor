---
title: Endlessh Configuration
description: This script configures Endlessh by applying the configuration stored in `${XDG_DATA_HOME:-$HOME/.ssh}/endlessh/config` if the `endlessh` application is available
sidebar_label: 30 Endlessh Configuration
slug: /scripts/after/run_onchange_after_30-endlessh.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_30-endlessh.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_30-endlessh.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_30-endlessh.sh.tmpl
---
# Endlessh Configuration

This script configures Endlessh by applying the configuration stored in `${XDG_DATA_HOME:-$HOME/.ssh}/endlessh/config` if the `endlessh` application is available

## Overview

Endlessh is a endless SSH tarpit that slowly shows an infinitely long SSH welcome banner on the default
SSH port. It is intended to break unsophisticated malware that targets SSH.

If the `endlessh` program is installed, this script applies the configuration stored in `home/private_dot_ssh/endlessh/config.tmpl`
(that unpacks with Chezmoi to `~/.ssh/endlessh/config`) to the system location and then starts the service.

**Note:** _This script runs under the assumption that the actual SSH port which is defined in `home/.chezmoidata.yaml`
is assigned to a non-standard port like 2214. This allows the default port to be used for `endlessh`._

## Links

* [Endlessh GitHub repository](https://github.com/skeeto/endlessh)
* [Endlessh configuration](https://github.com/megabyte-labs/install.doctor/blob/master/home/private_dot_ssh/endlessh/config.tmpl)



## Source Code

```
{{- if eq .host.distro.family "linux" -}}
#!/usr/bin/env bash
# @file Endlessh Configuration
# @brief Applies the Endlessh configuration and starts the service on Linux systems
# @description
#     Endlessh is a endless SSH tarpit that slowly shows an infinitely long SSH welcome banner on the default
#     SSH port. It is intended to break unsophisticated malware that targets SSH.
#
#     If the `endlessh` program is installed, this script applies the configuration stored in `home/private_dot_ssh/endlessh/config.tmpl`
#     (that unpacks with Chezmoi to `~/.ssh/endlessh/config`) to the system location and then starts the service.
#
#     **Note:** _This script runs under the assumption that the actual SSH port which is defined in `home/.chezmoidata.yaml`
#     is assigned to a non-standard port like 2214. This allows the default port to be used for `endlessh`._
#
#     ## Links
#
#     * [Endlessh GitHub repository](https://github.com/skeeto/endlessh)
#     * [Endlessh configuration](https://github.com/megabyte-labs/install.doctor/blob/master/home/private_dot_ssh/endlessh/config.tmpl)

# @file Endlessh Configuration
# @brief This script configures Endlessh by applying the configuration stored in `${XDG_DATA_HOME:-$HOME/.ssh}/endlessh/config` if the `endlessh` application is available
# @description
#     This script applies the Endlessh configuration stored in `${XDG_DATA_HOME:-$HOME/.ssh}/endlessh/config` if endlessh is installed.
#     Endlessh is and SSH Tarpit configured to listen for incoming connection on the given port and respond slowly with a random, endless SSH banner. To protect the real server,
#     configure Endlessh to listen on the default SSH port (22), while the real server listens to a different port.
#
#     ## Configuration Variables
#
#     The following chart details the input variable(s) that are used to determine the configuration of the endlessh:
#
#     | Variable        | Description                                                |
#     |-----------------|------------------------------------------------------------|
#     | `endlesshPort`  | The port that endlessh listens to for incoming connections |
#
#     ## Links
#
#     * [Default Endlessh configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/private_dot_ssh/endlessh/config.tmpl)
#     * [Secrets / Environment variables documentation](https://install.doctor/docs/customization/secrets)

# endlessh config hash: {{- include (joinPath .host.home ".ssh" "endlessh" "config") | sha256sum -}}

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Configures endlessh service
function configureEndlessh() {
    ### Update the service configuration file
    logg info 'Updating `endlessh` service configuration file'
    sudo sed -i 's/^.*#AmbientCapabilities=CAP_NET_BIND_SERVICE/AmbientCapabilities=CAP_NET_BIND_SERVICE/' /usr/lib/systemd/system/endlessh.service
    sudo sed -i 's/^.*PrivateUsers=true/#PrivateUsers=true/' /usr/lib/systemd/system/endlessh.service
    logg info 'Reloading systemd'
    sudo systemctl daemon-reload

    ### Update capabilities of `endlessh`
    logg info 'Updating capabilities of `endlessh`'
    sudo setcap 'cap_net_bind_service=+ep' /usr/bin/endlessh

    ### Restart / enable Endlessh
    logg info 'Enabling the `endlessh` service'
    sudo systemctl enable endlessh
    logg info 'Restarting the `endlessh` service'
    sudo systemctl restart endlessh
}

### Update /etc/endlessh/config if environment is not WSL
if [[ ! "$(test -d proc && grep Microsoft /proc/version > /dev/null)" ]]; then
    if command -v endlessh > /dev/null; then
        if [ -d /etc/endlessh ]; then
            logg info 'Copying ~/.ssh/endlessh/config to /etc/endlessh/config'
            sudo cp -f "$HOME/.ssh/endlessh/config" /etc/endlessh/config

            configureEndlessh || CONFIGURE_EXIT_CODE=$?
            if [ -n "$CONFIGURE_EXIT_CODE" ]; then
                logg error 'Configuring `endlessh` service failed' && exit 1
            else
                logg success 'Successfully configured `endlessh` service'
            fi
        elif [ -f /etc/endlessh.conf ]; then
            logg info 'Copying ~/.ssh/endlessh/config to /etc/endlessh.conf'
            sudo cp -f "$HOME/.ssh/endlessh/config" /etc/endlessh.conf

            configureEndlessh || CONFIGURE_EXIT_CODE=$?
            if [ -n "$CONFIGURE_EXIT_CODE" ]; then
                logg error 'Configuring `endlessh` service failed' && exit 1
            else
                logg success 'Successfully configured `endlessh` service'
            fi
        else
            logg warn 'Neither the /etc/endlessh folder nor the /etc/endlessh.conf file exist'
        fi
    else
        logg info 'Skipping Endlessh configuration because the `endlessh` executable is not available in the PATH'
    fi
else
    logg info 'Skipping Endlessh configuration since environment is WSL'
fi

{{ end -}}
```
