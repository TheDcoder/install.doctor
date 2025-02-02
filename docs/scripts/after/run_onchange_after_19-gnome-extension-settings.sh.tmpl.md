---
title: GNOME Extension Settings
description: Applies GNOME extension settings from various data sources.
sidebar_label: 19 GNOME Extension Settings
slug: /scripts/after/run_onchange_after_19-gnome-extension-settings.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_19-gnome-extension-settings.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_19-gnome-extension-settings.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_19-gnome-extension-settings.sh.tmpl
---
# GNOME Extension Settings

Applies GNOME extension settings from various data sources.

## Overview

This script ensures your GNOME extensions come pre-configured to your liking. It provides the ability
to automatically install a configurable list of GNOME extensions as well as apply your preferred settings.



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file GNOME Extension Settings
# @brief Applies GNOME extension settings from various data sources.
# @description
#     This script ensures your GNOME extensions come pre-configured to your liking. It provides the ability
#     to automatically install a configurable list of GNOME extensions as well as apply your preferred settings.

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

# extension config hash: {{ include (joinPath .host.home ".config" "desktop" "gnome.yml") | sha256sum }}

### Ensure /tmp/install-gnome-extensions.txt is not present on the system
if [ -f /tmp/install-gnome-extensions.txt ]; then
    rm -f /tmp/install-gnome-extensions.txt
fi

### Register temporary file for gnome.yml JSON
if [ -f "$HOME/.config/desktop/gnome.yml" ]; then
    TMP_YQ="$(mktemp)"
    cat "$HOME/.config/desktop/gnome.yml" | yq e -o=j '.' > "$TMP_YQ"
fi

### Populate /tmp/install-gnome-extensions.txt with GNOME extensions that need to be installed
if [ -f "$HOME/.config/desktop/gnome.yml" ]; then
    cat "$TMP_YQ" | jq -c '.default_gnome_extensions[] | tojson' | while read EXT; do
        TMP="$(mktemp)"
        echo "$EXT" | sed 's/^.\(.*\).$/\1/' > "$TMP"
        EXT_URL="$(cat "$TMP" | jq -r '.url')"
        EXT_ID="$(cat "$TMP" | jq -r '.regex')"
        echo "$EXT_URL" >> /tmp/install-gnome-extensions.txt
        if [ ! -d "${XDG_DATA_HOME:-$HOME/.local/share}/gnome-shell/extensions" ]; then
            mkdir -p "${XDG_DATA_HOME:-$HOME/.local/share}/gnome-shell/extensions"
        fi
        find "${XDG_DATA_HOME:-$HOME/.local/share}/gnome-shell/extensions" -mindepth 1 -maxdepth 1 -type d | while read EXT_FOLDER; do
            if [[ "$EXT_FOLDER" == *"$EXT_ID"* ]] && [ -f /tmp/install-gnome-extensions.txt ]; then
                TMP_EXT="$(mktemp)"
                head -n -1 /tmp/install-gnome-extensions.txt > "$TMP_EXT"
                mv -f "$TMP_EXT" /tmp/install-gnome-extensions.txt > /dev/null
            fi
        done
    done
else
    logg warn 'The `~/.config/desktop/gnome.yml` file is missing so GNOME extension install orders cannot be calculated'
fi

### Remove /tmp/install-gnome-extensions.txt if it is empty
if [ "$(cat /tmp/install-gnome-extensions.txt)" == "" ]; then
    rm -f /tmp/install-gnome-extensions.txt > /dev/null
fi

### Install the GNOME extensions using the `install-gnome-extensions` script
if command -v install-gnome-extensions > /dev/null; then
    if [ -f /tmp/install-gnome-extensions.txt ]; then
        logg info 'Running the `install-gnome-extensions` script'
        cd /tmp
        install-gnome-extensions --enable --overwrite --file /tmp/install-gnome-extensions.txt
        rm -f /tmp/install-gnome-extensions.txt
        logg success 'Finished installing the GNOME extensions'
    else
        logg info 'No new GNOME extensions to install'
    fi
else
    logg warn 'Cannot install GNOME extensions because the `install-gnome-extensions` script is missing from ~/.local/bin'
fi

### Apply plugin gsettings
if [ -f "$HOME/.config/desktop/gnome.yml" ]; then
    cat "$TMP_YQ" | jq -c '.default_gnome_extensions[] | tojson' | while read EXT; do
        if [ "$DEBUG_MODE" == 'true' ]; then
            logg info 'Extension data:'
            echo "$EXT"
        fi
        TMP="$(mktemp)"
        echo "$EXT" | sed 's/^.\(.*\).$/\1/' > "$TMP"
        EXT_URL="$(cat "$TMP" | jq -r '.url')"
        EXT_ID="$(cat "$TMP" | jq -r '.regex')"
        if [ "$DEBUG_MODE" == 'true' ]; then
            logg info 'Extension ID:'
            echo "$EXT_ID"
        fi
        EXT_SETTINGS_TYPE="$(cat "$TMP" | jq -r '.settings | type')"
        EXT_SETTINGS="$(cat "$TMP" | jq -r '.settings')"
        if [ "$EXT_SETTINGS" != 'null' ]; then
            logg info 'Evaluating extension settings for `'"$EXT_ID"'`'
            if [ "$EXT_SETTINGS_TYPE" == 'array' ]; then
                cat "$TMP" | jq -r '.settings[]' | while read EXT_SETTING; do
                    logg info 'Applying following extension setting:'
                    echo "$EXT_SETTING"
                    eval "$EXT_SETTING"
                done
            else
                logg info 'Applying following extension setting:'
                echo "$EXT_SETTINGS"
                eval "$EXT_SETTINGS"
            fi
            logg success 'Applied gsettings configuration for the `'"$EXT_ID"'` GNOME extension'
        fi
    done
fi

{{- end }}
```
