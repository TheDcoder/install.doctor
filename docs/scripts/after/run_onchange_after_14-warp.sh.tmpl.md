---
title: CloudFlare WARP
description: Installs CloudFlare WARP, ensures proper security certificates are in place, and connects the device to CloudFlare WARP.
sidebar_label: 14 CloudFlare WARP
slug: /scripts/after/run_onchange_after_14-warp.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_14-warp.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_14-warp.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_14-warp.sh.tmpl
---

# CloudFlare WARP

Installs CloudFlare WARP, ensures proper security certificates are in place, and connects the device to CloudFlare WARP.

## Overview

This script is intended to connect the device to CloudFlare's Zero Trust network with nearly all of its features unlocked.
Homebrew is used to install the `warp-cli` on macOS. On Linux, it can install `warp-cli` on most Debian systems and some RedHat
systems. CloudFlare WARP's [download page](https://pkg.cloudflareclient.com/packages/cloudflare-warp) is somewhat barren.

## MDM Configuration

If CloudFlare WARP successfully installs, it first applies MDM configurations (managed configurations). If you would like CloudFlare
WARP to connect completely headlessly (while losing some "user-posture" settings), then you can populate the following two secrets:

1. `CLOUDFLARE_TEAMS_CLIENT_ID` - The ID from a CloudFlare Teams service token. See [this article](https://developers.cloudflare.com/cloudflare-one/identity/service-tokens/).
2. `CLOUDFLARE_TEAMS_CLIENT_SECRET` - The secret from a CloudFlare Teams service token.

The two variables above can be passed in using either of the methods described in the [Secrets documentation](https://install.doctor/docs/customization/secrets).

## Headless CloudFlare WARP Connection

Even if you do not provide the two variables mentioned above, the script will still headlessly connect your device to the public CloudFlare WARP
network, where you will get some of the benefits of a VPN for free. Otherwise, if they were passed in, then the script
finishes by connecting to CloudFlare Teams.

## Notes

According to CloudFlare Teams [documentation on MDM deployment](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/mdm-deployment/),
on macOS the `com.cloudflare.warp.plist` file gets erased on reboot. Also, according to the documentation, the only way around this is to leverage
an MDM SaaS provider like JumpCloud.

## Links

- [Linux managed configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/warp/private_mdm.xml.tmpl)
- [macOS managed configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/Library/Managed%20Preferences/private_com.cloudflare.warp.plist.tmpl)

## Source Code

```
{{- if (ne .host.distro.family "windows") -}}
#!/usr/bin/env bash
# @file CloudFlare WARP
# @brief Installs CloudFlare WARP, ensures proper security certificates are in place, and connects the device to CloudFlare WARP.
# @description
#     This script is intended to connect the device to CloudFlare's Zero Trust network with nearly all of its features unlocked.
#     Homebrew is used to install the `warp-cli` on macOS. On Linux, it can install `warp-cli` on most Debian systems and some RedHat
#     systems. CloudFlare WARP's [download page](https://pkg.cloudflareclient.com/packages/cloudflare-warp) is somewhat barren.
#
#     ## MDM Configuration
#
#     If CloudFlare WARP successfully installs, it first applies MDM configurations (managed configurations). If you would like CloudFlare
#     WARP to connect completely headlessly (while losing some "user-posture" settings), then you can populate the following two secrets:
#
#     1. `CLOUDFLARE_TEAMS_CLIENT_ID` - The ID from a CloudFlare Teams service token. See [this article](https://developers.cloudflare.com/cloudflare-one/identity/service-tokens/).
#     2. `CLOUDFLARE_TEAMS_CLIENT_SECRET` - The secret from a CloudFlare Teams service token.
#
#     The two variables above can be passed in using either of the methods described in the [Secrets documentation](https://install.doctor/docs/customization/secrets).
#
#     ## Headless CloudFlare WARP Connection
#
#     Even if you do not provide the two variables mentioned above, the script will still headlessly connect your device to the public CloudFlare WARP
#     network, where you will get some of the benefits of a VPN for free. Otherwise, if they were passed in, then the script
#     finishes by connecting to CloudFlare Teams.
#
#     ## Notes
#
#     According to CloudFlare Teams [documentation on MDM deployment](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/mdm-deployment/),
#     on macOS the `com.cloudflare.warp.plist` file gets erased on reboot. Also, according to the documentation, the only way around this is to leverage
#     an MDM SaaS provider like JumpCloud.
#
#     ## Links
#
#     * [Linux managed configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/warp/private_mdm.xml.tmpl)
#     * [macOS managed configuration](https://github.com/megabyte-labs/install.doctor/tree/master/home/Library/Managed%20Preferences/private_com.cloudflare.warp.plist.tmpl)

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Install CloudFlare WARP (on non-WSL *nix systems)
if [[ ! "$(test -d /proc && grep Microsoft /proc/version > /dev/null)" ]]; then
    if [ -d /System ] && [ -d /Applications ]; then
        ### Install on macOS
        brew install --cask cloudflare-warp
    elif [ '{{ .host.distro.id }}' = 'debian' ]; then
        ### Add CloudFlare WARP desktop app apt-get source
        if [ ! -f /etc/apt/sources.list.d/cloudflare-client.list ]; then
            logg info 'Adding CloudFlare WARP keyring'
            curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
            logg info 'Adding apt source reference'
            echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
        fi

        ### Update apt-get and install the CloudFlare WARP CLI
        sudo apt-get update && sudo apt-get install -y cloudflare-warp
    elif [ '{{ .host.distro.id }}' = 'ubuntu' ]; then
        ### Add CloudFlare WARP desktop app apt-get source
        if [ ! -f /etc/apt/sources.list.d/cloudflare-client.list ]; then
            logg info 'Adding CloudFlare WARP keyring'
            curl https://pkg.cloudflareclient.com/pubkey.gpg | sudo gpg --yes --dearmor --output /usr/share/keyrings/cloudflare-warp-archive-keyring.gpg
            logg info 'Adding apt source reference'
            echo "deb [arch=amd64 signed-by=/usr/share/keyrings/cloudflare-warp-archive-keyring.gpg] https://pkg.cloudflareclient.com/ $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflare-client.list
        fi

        ### Update apt-get and install the CloudFlare WARP CLI
        sudo apt-get update && sudo apt-get install -y cloudflare-warp
    elif command -v dnf > /dev/null && command -v rpm > /dev/null; then
        ### This is made for CentOS 8 and works on Fedora 36 (hopefully 36+ as well) with `nss-tools` as a dependency
        sudo dnf instal -y nss-tools || NSS_TOOL_EXIT=$?
        if [ -n "$NSS_TOOL_EXIT" ]; then
            logg warn 'Unable to install `nss-tools` which was a requirement on Fedora 36 and assumed to be one on other systems as well.'
        fi
        ### According to the download site, this is the only version available for RedHat-based systems
        sudo rpm -ivh https://pkg.cloudflareclient.com/cloudflare-release-el8.rpm || RPM_EXIT_CODE=$?
        if [ -n "$RPM_EXIT_CODE" ]; then
            logg error 'Unable to install CloudFlare WARP using RedHat 8 RPM package'
        fi
    fi
fi


### Ensure certificate is installed
### TODO: Ensure duplicate certificates are not stored in these files below
# Source: https://developers.cloudflare.com/cloudflare-one/static/documentation/connections/Cloudflare_CA.crt
# Source: https://developers.cloudflare.com/cloudflare-one/static/documentation/connections/Cloudflare_CA.pem
if [ -d /System ] && [ -d /Applications ] && command -v warp-cli > /dev/null; then
    ### Ensure certificate installed on macOS
    sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain "$HOME/.local/etc/ssl/cloudflare/Cloudflare_CA.crt"
    if [ -f /usr/local/etc/ca-certificates/cert.pem ]; then
        echo | sudo cat - "$HOME/.local/etc/ssl/cloudflare/Cloudflare_CA.pem" >> /usr/local/etc/ca-certificates/cert.pem
    else
        logg error 'Unable to add `Cloudflare_CA.pem` because `/usr/local/etc/ca-certificates/cert.pem` does not exist!' && exit 1
    fi
fi

if command -v warp-cli > /dev/null; then
    ### Ensure MDM settings are applied (deletes after reboot on macOS)
    ### TODO: Ensure `.plist` can be added to `~/Library/Managed Preferences` and not just `/Library/Managed Preferences`
    # Source: https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/mdm-deployment/
    # Source for JumpCloud: https://developers.cloudflare.com/cloudflare-one/static/documentation/connections/CloudflareWARP.mobileconfig
    if [ -d /System ] && [ -d /Applications ]; then
        sudo cp -f "$HOME/Library/Managed Preferences/com.cloudflare.warp.plist" '/Library/Managed Preferences/com.cloudflare.warp.plist'
        sudo plutil -convert binary1 '/Library/Managed Preferences/com.cloudflare.warp.plist'
    elif [ -f "${XDG_CONFIG_HOME:-$HOME/.config}/warp/mdm.xml" ]; then
        sudo mkdir -p /var/lib/cloudflare-warp
        sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/warp/mdm.xml" /var/lib/cloudflare-warp/mdm.xml
    fi

    ### Register CloudFlare WARP
    if warp-cli --accept-tos status | grep 'Registration missing' > /dev/null; then
        logg info 'Registering CloudFlare WARP'
        warp-cli --accept-tos register
    else
        logg info 'Already registered with CloudFlare WARP'
    fi

    ### Connect CloudFlare WARP
    if warp-cli --accept-tos status | grep 'Disconnected' > /dev/null; then
        logg info 'Connecting to CloudFlare WARP'
        warp-cli --accept-tos connect
    else
        logg info 'Already connected to CloudFlare WARP'
    fi
else
    logg warn '`warp-cli` was not installed so CloudFlare Zero Trust cannot be joined'
fi
{{ end -}}
```
