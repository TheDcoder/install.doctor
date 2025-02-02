---
title: Tor Configuration
description: This script applies the Tor configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/tor/torrc` to the system and then restarts Tor
sidebar_label: 27 Tor Configuration
slug: /scripts/after/run_onchange_after_27-tor.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_27-tor.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_27-tor.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_27-tor.sh.tmpl
---
# Tor Configuration

This script applies the Tor configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/tor/torrc` to the system and then restarts Tor

## Overview

Tor is a network that uses onion routing, originally published by the US Navy. It is leveraged by privacy enthusiasts
and other characters that deal with sensitive material, like journalists and people buying drugs on the internet.
This script:

1. Determines the system configuration file location
2. Applies the configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/tor/torrc`
3. Enables and restarts the Tor service with the new configuration

## Links

* [Tor configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/tor/torrc)



## Source Code

```
{{- if and (ne .host.distro.family "windows") (ne .host.work true) -}}
#!/usr/bin/env bash
# @file Tor Configuration
# @brief This script applies the Tor configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/tor/torrc` to the system and then restarts Tor
# @description
#     Tor is a network that uses onion routing, originally published by the US Navy. It is leveraged by privacy enthusiasts
#     and other characters that deal with sensitive material, like journalists and people buying drugs on the internet.
#     This script:
#
#     1. Determines the system configuration file location
#     2. Applies the configuration stored at `${XDG_CONFIG_HOME:-HOME/.config}/tor/torrc`
#     3. Enables and restarts the Tor service with the new configuration
#
#     ## Links
#
#     * [Tor configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/tor/torrc)

# tor config hash: {{ include (joinPath .host.home ".config" "tor" "torrc") | sha256sum }}

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

# @description Determine the Tor configuration location by checking whether the system is macOS or Linux
if [ -d /Applications ] && [ -d /System ]; then
    # macOS
    TORRC_CONFIG_DIR=/usr/local/etc/tor
else
    # Linux
    TORRC_CONFIG_DIR=/etc/tor
fi
TORRC_CONFIG="$TORRC_CONFIG_DIR/torrc"

# @description Apply the configuration if the `torrc` binary is available in the `PATH`
if command -v toron > /dev/null; then
    if [ -d  "$TORRC_CONFIG_DIR" ]; then
        # @description Copy the configuration from `${XDG_CONFIG_HOME:-$HOME/.config}/tor/torrc` to the system configuration file location
        sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/tor/torrc" "$TORRC_CONFIG"
        sudo chmod 600 "$TORRC_CONFIG"

        # @description Enable and restart the Tor service
        if [ -d /Applications ] && [ -d /System ]; then
            # macOS
            brew services restart tor
        else
            if [[ ! "$(test -d /proc && grep Microsoft /proc/version > /dev/null)" ]]; then
                # Linux
                sudo systemctl enable tor
                sudo systemctl restart tor
            else
                logg info 'Environment is WSL so the Tor systemd service will not be enabled / restarted'
            fi
        fi
    else
        logg warn 'The '"$TORRC_CONFIG_DIR"' directory is missing'
    fi
else
    logg warn '`toron` is missing from the PATH'
fi

{{ end -}}
```
