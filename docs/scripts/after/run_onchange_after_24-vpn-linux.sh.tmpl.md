---
title: Linux OpenVPN / WireGuard Profiles
description: Installs both OpenVPN and WireGuard VPN profiles on Linux devices.
sidebar_label: 24 Linux OpenVPN / WireGuard Profiles
slug: /scripts/after/run_onchange_after_24-vpn-linux.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_24-vpn-linux.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_24-vpn-linux.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_24-vpn-linux.sh.tmpl
---
# Linux OpenVPN / WireGuard Profiles

Installs both OpenVPN and WireGuard VPN profiles on Linux devices.

## Overview

This script installs OpenVPN and WireGuard VPN profiles. It does a few things to install the profiles and make sure
they are usable by desktop users:

1. It ensures OpenVPN and `NetworkManager-*` plugins are installed (this allows you to see all the different VPN profile types available when you try to import a VPN profile on Linux devices)
2. Imports the OpenVPN profiles stored in `${XDG_CONFIG_HOME:-$HOME/.config}/vpn`
3. Applies the OpenVPN username and password to all the OpenVPN profiles (which can be passed in as `OVPN_USERNAME` and `OVPN_PASSWORD` if you use the environment variable method)
4. Bypasses the OpenVPN connection for all the networks defined in `.host.vpn.excludedSubnets` (in the `home/.chezmoi.yaml.tmpl` file)
5. Repeats the process for WireGuard by looping through all the `*.nmconnection` files stored in `${XDG_CONFIG_HOME:-$HOME/.config}/vpn` (username and password should already be stored in the encrypted files)

## Creating VPN Profiles

More details on embedding your VPN profiles into your Install Doctor fork can be found by reading the [Secrets documentation](https://install.doctor/docs/customization/secrets#vpn-profiles).

## Links

* [VPN profile folder](https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/vpn)
* [VPN profile documentation](https://install.doctor/docs/customization/secrets#vpn-profiles)



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file Linux OpenVPN / WireGuard Profiles
# @brief Installs both OpenVPN and WireGuard VPN profiles on Linux devices.
# @description
#     This script installs OpenVPN and WireGuard VPN profiles. It does a few things to install the profiles and make sure
#     they are usable by desktop users:
#
#     1. It ensures OpenVPN and `NetworkManager-*` plugins are installed (this allows you to see all the different VPN profile types available when you try to import a VPN profile on Linux devices)
#     2. Imports the OpenVPN profiles stored in `${XDG_CONFIG_HOME:-$HOME/.config}/vpn`
#     3. Applies the OpenVPN username and password to all the OpenVPN profiles (which can be passed in as `OVPN_USERNAME` and `OVPN_PASSWORD` if you use the environment variable method)
#     4. Bypasses the OpenVPN connection for all the networks defined in `.host.vpn.excludedSubnets` (in the `home/.chezmoi.yaml.tmpl` file)
#     5. Repeats the process for WireGuard by looping through all the `*.nmconnection` files stored in `${XDG_CONFIG_HOME:-$HOME/.config}/vpn` (username and password should already be stored in the encrypted files)
#
#     ## Creating VPN Profiles
#
#     More details on embedding your VPN profiles into your Install Doctor fork can be found by reading the [Secrets documentation](https://install.doctor/docs/customization/secrets#vpn-profiles).
#
#     ## Links
#
#     * [VPN profile folder](https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/vpn)
#     * [VPN profile documentation](https://install.doctor/docs/customization/secrets#vpn-profiles)

{{ $ovpnUsername := (env "OVPN_USERNAME") }}
{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "OVPN_USERNAME")) }}
{{   $ovpnUsername := (includeTemplate "secrets/OVPN_USERNAME" | decrypt) }}
{{ end }}

{{ $ovpnPassword := (env "OVPN_PASSWORD") }}
{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "OVPN_PASSWORD")) }}
{{   $ovpnPassword := (includeTemplate "secrets/OVPN_PASSWORD" | decrypt) }}
{{ end }}

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

RESTART_NM=false

# @description Ensure `NetworkManager` plugins are
# NOTE: By default, all the NetworkManager plugins are installed.
if command -v apt-get > /dev/null; then
    sudo apt-get install -y network-manager*
elif command -v dnf > /dev/null; then
    sudo dnf install -y openvpn NetworkManager*
elif command -v pacman > /dev/null; then
    sudo pacman -Syu openvpn networkmanager*
else
    logg warn 'Unknown package manager - install OpenVPN / WireGuard / NetworkManager plugins individually'
fi

function ensureNetworkConfigs() {
    if [ ! -d /etc/network/if-up.d ]; then
        logg info 'Creating /etc/network/if-up.d folder'
        sudo mkdir -p /etc/network/if-up.d
    fi
    if [ ! -d /etc/network/if-post-down.d ]; then
        logg info 'Creating /etc/network/if-post.d folder'
        sudo mkdir -p /etc/network/if-post.d
    fi
}

# @description Ensures `nmcli` (the CLI for NetworkManager) is available in the `PATH`
if command -v nmcli > /dev/null; then
    # @description Sets up OpenVPN profiles
    if [ '{{ $ovpnUsername }}' != '' ] && [ '{{ $ovpnPassword }}' != '' ]; then
        find "${XDG_CONFIG_HOME:-$HOME/.config}/vpn" -type f -name "*.ovpn" | while read OVPN_FILE; do
            # @description Adds the OpenVPN profiles by importing the `*.ovpn` files in `${XDG_CONFIG_HOME:-$HOME/.config}/vpn` and then applying the OpenVPN username and password
            logg info "Adding $OVPN_FILE to NetworkManager OpenVPN profiles"
            OVPN_NAME="$(basename "$OVPN_FILE" | sed 's/.ovpn$//')"
            nmcli connection import type openvpn file "$OVPN_FILE"
            nmcli connection modify "$OVPN_NAME" +vpn.data 'username={{- $ovpnUsername }}'
            nmcli connection modify "$OVPN_NAME" vpn.secrets 'password={{- $ovpnPassword }}'
            nmcli connection modify "$OVPN_NAME" +vpn.data password-flags=0

            # @description Register the excluded subnets in the routeadd / routedel files
            for EXCLUDED_SUBNET in '{{ $removeShortcuts := join "' '" .host.vpn.excludedSubnets }}'; do
                ensureNetworkConfigs
                nmcli connection modify "$OVPN_NAME" +ipv4.routes "$EXCLUDED_SUBNET" | sudo tee -a /etc/network/if-up.d/routeadd
                nmcli connection modify "$OVPN_NAME" -ipv4.routes "$EXCLUDED_SUBNET" | sudo tee -a /etc/network/if-post-down.d/routedel
            fi
            RESTART_NM=true
        done
    else
        logg info 'Either the OpenVPN username or password is undefined.'
        logg info 'See the `docs/VARIABLES.md` file for details.'
    fi

{{ if (stat (joinPath .host.home ".config" "age" "chezmoi.txt")) }}
    # @description Setup WireGuard profiles
    if [ -d /etc/NetworkManager/system-connections ]; then
        find "${XDG_CONFIG_HOME:-$HOME/.config}/vpn" -type f -name "*.nmconnection" | while read WG_FILE; do
            # @description Ensure the WireGuard NetworkManager plugin is available
            if [ ! -d /usr/lib/NetworkManager/nm-wireguard-service ]; then
                logg info 'The `nm-wireguard-service` is not present'
                logg info 'Installing the `nm-wireguard-service`'
            fi

            # @description Add the WireGuard profiles
            logg info "Adding $WG_FILE to /etc/NetworkManager/system-connections
            WG_FILENAME="$(basename "$WG_FILE")"
            chezmoi decrypt "$WG_FILE" | sudo tee "/etc/NetworkManager/system-connections/$WG_FILENAME"

            # @description Register the excluded subnets in the routeadd / routedel files
            for EXCLUDED_SUBNET in '{{ $removeShortcuts := join "' '" .host.vpn.excludedSubnets }}'; do
                ensureNetworkConfigs
                WG_PROFILE_NAME="$(echo "$WG_FILENAME" | sed 's/.nmconnection$//')"
                nmcli connection modify "$WG_PROFILE_NAME" +ipv4.routes "$EXCLUDED_SUBNET" | sudo tee -a /etc/network/if-up.d/routeadd
                nmcli connection modify "$WG_PROFILE_NAME" -ipv4.routes "$EXCLUDED_SUBNET" | sudo tee -a /etc/network/if-post-down.d/routedel
            fi
            RESTART_NM=true
        done
    else
        logg warn '/etc/NetworkManager/system-connections is not a directory!'
    fi
{{ end -}}

    # @description Restart NetworkManager if changes were made and environment is not WSL
    if [ "$RESTART_NM" == 'true' ] && [[ ! "$(test -d proc && grep Microsoft /proc/version > /dev/null)" ]]; then
        logg info 'Restarting NetworkManager since VPN profiles were updated'
        sudo service NetworkManager restart
    fi
else
    logg warn '`nmcli` is unavailable'
fi

{{ end -}}```
