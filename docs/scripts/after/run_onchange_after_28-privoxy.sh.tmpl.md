---
title: Privoxy Configuration
description: This script applies the Privoxy configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/privoxy/config` to the system and then restarts Privoxy
sidebar_label: 28 Privoxy Configuration
slug: /scripts/after/run_onchange_after_28-privoxy.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_28-privoxy.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_28-privoxy.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_28-privoxy.sh.tmpl
---
# Privoxy Configuration

This script applies the Privoxy configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/privoxy/config` to the system and then restarts Privoxy

## Overview

Privoxy is a web proxy that can be combined with Tor to provide an HTTPS / HTTP proxy that can funnel all traffic
through Tor. This script:

1. Determines the system configuration file location
2. Applies the configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/privoxy/config`
3. Enables and restarts the Privoxy service with the new configuration

## Links

* [Privoxy configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/privoxy/config)



## Source Code

```
{{- if (ne .host.distro.family "windows") -}}
#!/usr/bin/env bash
# @file Privoxy Configuration
# @brief This script applies the Privoxy configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/privoxy/config` to the system and then restarts Privoxy
# @description
#     Privoxy is a web proxy that can be combined with Tor to provide an HTTPS / HTTP proxy that can funnel all traffic
#     through Tor. This script:
#
#     1. Determines the system configuration file location
#     2. Applies the configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/privoxy/config`
#     3. Enables and restarts the Privoxy service with the new configuration
#
#     ## Links
#
#     * [Privoxy configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/privoxy/config)

# privoxy config hash: {{ include (joinPath .host.home ".config" "privoxy" "config") | sha256sum }}

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

# @description Define the Privoxy configuration location based on whether system is macOS or Linux
if [ -d /Applications ] && [ -d /System ]; then
    # macOS
    PRIVOXY_CONFIG_DIR=/usr/local/etc/privoxy
else
    # Linux
    PRIVOXY_CONFIG_DIR=/etc/privoxy
fi
PRIVOXY_CONFIG="$PRIVOXY_CONFIG_DIR/config"

# @description Copy Privoxy configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/privoxy/config` to the system location
if command -v privoxy > /dev/null; then
    if [ -d  "$PRIVOXY_CONFIG_DIR" ]; then
        sudo cp -f "${XDG_CONFIG_HOME:-HOME/.config}/privoxy/config" "$PRIVOXY_CONFIG"
        sudo chmod 600 "$PRIVOXY_CONFIG"
        sudo chown privoxy:privoxy "$PRIVOXY_CONFIG"

        # @description Restart Privoxy after configuration is applied
        if [ -d /Applications ] && [ -d /System ]; then
            # macOS
            brew services restart privoxy
        else
            if [[ ! "$(test -d /proc && grep Microsoft /proc/version > /dev/null)" ]]; then
                # Linux
                sudo systemctl enable privoxy
                sudo systemctl restart privoxy
            else
                logg info 'The system is a WSL environment so the Privoxy systemd service will not be enabled / restarted'
            fi
        fi
    else
        logg warn 'The '"$PRIVOXY_CONFIG_DIR"' directory is missing'
    fi
else
    logg warn '`privoxy` is missing from the PATH'
fi

{{ end -}}
```
