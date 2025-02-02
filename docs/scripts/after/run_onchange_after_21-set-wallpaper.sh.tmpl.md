---
title: macOS Set Wallpaper
description: Ensures the macOS wallpaper is set to the Betelgeuse wallpaper for macOS.
sidebar_label: 21 macOS Set Wallpaper
slug: /scripts/after/run_onchange_after_21-set-wallpaper.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_21-set-wallpaper.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_21-set-wallpaper.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_21-set-wallpaper.sh.tmpl
---
# macOS Set Wallpaper

Ensures the macOS wallpaper is set to the Betelgeuse wallpaper for macOS.

## Overview

This script ensures the macOS desktop wallpaper is set to the macOS Betelgeuse wallpaper. It uses the
`m` CLI to apply the change.



## Source Code

```
{{- if (ne .host.distro.family "darwin") -}}
#!/usr/bin/env bash
# @file macOS Set Wallpaper
# @brief Ensures the macOS wallpaper is set to the Betelgeuse wallpaper for macOS.
# @description
#     This script ensures the macOS desktop wallpaper is set to the macOS Betelgeuse wallpaper. It uses the
#     `m` CLI to apply the change.

# Betelgeuse-macOS wallpaper hash: {{ include (joinPath .chezmoi.homeDir ".local" "src" "betelgeuse" "share" "wallpapers" "Betelgeuse-macOS" "contents" "source.png") | sha256sum }}

### Set macOS wallpaper
if command -v m > /dev/null && [ -f "$HOME/.local/src/betelgeuse/share/wallpapers/Betelgeuse-macOS/contents/source.png" ]; then
  m wallpaper "$HOME/.local/src/betelgeuse/share/wallpapers/Betelgeuse-macOS/contents/source.png"
else
  logg warn 'Either `m` or the macOS default wallpaper is missing.'
fi
{{ end -}}
```
