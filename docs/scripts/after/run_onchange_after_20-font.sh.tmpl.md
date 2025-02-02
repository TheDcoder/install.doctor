---
title: Linux Fonts
description: Ensures fonts are available at the system level and, on Linux, it configures the system font settings.
sidebar_label: 20 Linux Fonts
slug: /scripts/after/run_onchange_after_20-font.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_20-font.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_20-font.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_20-font.sh.tmpl
---
# Linux Fonts

Ensures fonts are available at the system level and, on Linux, it configures the system font settings.

## Overview

This script is utilized to ensure the same fonts are consistently used across the system.



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file Linux Fonts
# @brief Ensures fonts are available at the system level and, on Linux, it configures the system font settings.
# @description
#     This script is utilized to ensure the same fonts are consistently used across the system.

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

# font config hash: {{ include (joinPath .host.home ".config" "fontconfig" "fonts.conf") | sha256sum }}

### Sync user fonts with system fonts
if [ -d /Applications ] && [ -d /System ]; then
    # macOS
    logg info 'Copying fonts from ~/Library/Fonts and ~/.local/share/fonts to /Library/Fonts to make them available globally'
    FONT_DIR='/Library/Fonts'
    sudo rsync -av "$HOME/Library/Fonts" "$FONT_DIR"
    sudo rsync -av "$HOME/.local/share/fonts" "$FONT_DIR"
else
    # Linux
    logg info 'Copying fonts from ~/.local/share/fonts to /usr/local/share/fonts to make them available globally'
    FONT_DIR='/usr/local/share/fonts'
    sudo rsync -av "$HOME/.local/share/fonts" "$FONT_DIR"
fi

### Configure system font properties
if [ -d /etc/fonts ]; then
    logg info 'Copying ~/.config/fontconfig/fonts.conf to /etc/fonts/local.conf'
    sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/fontconfig/fonts.conf" /etc/fonts/local.conf
else
    logg warn 'The `/etc/fonts` directory is missing'
fi

{{ end -}}
```
