---
title: Tailscale
description: Connects the Tailscale client with the Tailscale network
sidebar_label: 59 Tailscale
slug: /scripts/after/run_onchange_after_59-connect-tailscale.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_59-connect-tailscale.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_59-connect-tailscale.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_59-connect-tailscale.sh.tmpl
---
# Tailscale

Connects the Tailscale client with the Tailscale network

## Overview

This script ensures the `tailscaled` system daemon is installed on macOS. Then, on both macOS and Linux, it connects to the Tailscale
network if the `TAILSCALE_AUTH_KEY` variable is provided.



## Source Code

```
{{- if or (and (ne .host.distro.family "windows") (stat (joinPath .host.home ".config" "age" "chezmoi.txt")) (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "TAILSCALE_AUTH_KEY")) (env "TAILSCALE_AUTH_KEY")) -}}
#!/usr/bin/env bash
# @file Tailscale
# @brief Connects the Tailscale client with the Tailscale network
# @description
#     This script ensures the `tailscaled` system daemon is installed on macOS. Then, on both macOS and Linux, it connects to the Tailscale
#     network if the `TAILSCALE_AUTH_KEY` variable is provided.

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Install the Tailscale system daemon
if [ -d /Applications ] && [ -d System ]; then
    # macOS
    if command -v tailscaled > /dev/null; then
        logg info 'Ensuring `tailscaled` system daemon is installed'
        sudo tailscaled install-system-daemon
        logg info '`tailscaled` system daemon is now installed and will load on boot'
    else
        logg info '`tailscaled` does not appear to be installed'
    fi
fi

### Connect to Tailscale network
TAILSCALE_AUTH_KEY="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "TAILSCALE_AUTH_KEY")) }}{{ includeTemplate "secrets/TAILSCALE_AUTH_KEY" | decrypt }}{{ else }}{{ env "TAILSCALE_AUTH_KEY" }}{{ end }}"
if command -v tailscale > /dev/null && [ "$TAILSCALE_AUTH_KEY" != "" ]; then
  logg info 'Connecting to Tailscale with user-defined authentication key'
  timeout 14 tailscale up --authkey="$TAILSCALE_AUTH_KEY" --accept-routes || EXIT_CODE=$?
  if [ -n "$EXIT_CODE" ]; then
    logg warn '`tailscale up` timed out'
  else
    logg success 'Connected to Tailscale network'
  fi
fi

{{- end -}}```
