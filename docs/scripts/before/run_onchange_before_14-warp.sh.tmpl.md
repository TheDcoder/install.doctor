---
title: CloudFlare WARP Repository
description: Adds the CloudFlare WARP `apt-get` repository to Debian and Ubuntu systems
sidebar_label: 14 CloudFlare WARP Repository
slug: /scripts/before/run_onchange_before_14-warp.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_14-warp.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_14-warp.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_14-warp.sh.tmpl
---
# CloudFlare WARP Repository

Adds the CloudFlare WARP `apt-get` repository to Debian and Ubuntu systems

## Overview

This script adds the CloudFlare WARP `apt-get` repository to Debian and Ubuntu systems. It currently does not support adding
repositories for other systems because they are not provided by CloudFlare.



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file CloudFlare WARP Repository
# @brief Adds the CloudFlare WARP `apt-get` repository to Debian and Ubuntu systems
# @description
#     This script adds the CloudFlare WARP `apt-get` repository to Debian and Ubuntu systems. It currently does not support adding
#     repositories for other systems because they are not provided by CloudFlare.

{{ includeTemplate "universal/logg-before" }}

if [ '{{ .host.distro.id }}' = 'debian' ]; then
    ### Add CloudFlare WARP desktop app apt-get source
    if [ ! -f /etc/apt/sources.list.d/cloudflare-client.list ]; then
        logg info 'Adding CloudFlare WARP keyring'
        curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

        logg info 'Adding apt source reference'
        echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

        sudo apt-get update
    fi
elif [ '{{ .host.distro.id }}' = 'ubuntu' ]; then
    ### Add CloudFlare WARP desktop app apt-get source
    if [ ! -f /etc/apt/sources.list.d/cloudflare-client.list ]; then
        logg info 'Adding CloudFlare WARP keyring'
        curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg

        logg info 'Adding apt source reference'
        echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list

        sudo apt-get update
    fi
fi
{{ end -}}
```
