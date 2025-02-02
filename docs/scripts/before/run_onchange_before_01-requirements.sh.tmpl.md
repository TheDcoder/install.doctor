---
title: Linux Required Dependencies
description: Ensures commonly used system packages that are common dependencies of other packages are installed
sidebar_label: 01 Linux Required Dependencies
slug: /scripts/before/run_onchange_before_01-requirements.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_01-requirements.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_01-requirements.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_01-requirements.sh.tmpl
---
# Linux Required Dependencies

Ensures commonly used system packages that are common dependencies of other packages are installed

## Overview

This script installs required system packages. Each system's required packages are defined in `home/.chezmoitemplates/$DISTRO_ID`,
where `$DISTRO_ID` is equal to the Linux distribution ID found in `/etc/os-release`.



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file Linux Required Dependencies
# @brief Ensures commonly used system packages that are common dependencies of other packages are installed
# @description
#     This script installs required system packages. Each system's required packages are defined in `home/.chezmoitemplates/$DISTRO_ID`,
#     where `$DISTRO_ID` is equal to the Linux distribution ID found in `/etc/os-release`.

# universal/common-dependencies hash: {{ include (joinPath ".chezmoitemplates" "universal" "common-dependencies") | sha256sum }}
# {{ .host.distro.id }}/common-dependencies hash: {{ include (joinPath ".chezmoitemplates" .host.distro.id "common-dependencies") | sha256sum }}

{{ includeTemplate "universal/logg-before" }}

{{- $packages := splitList " " (includeTemplate "universal/common-dependencies" .) -}}
{{- $additionalPackages := splitList " " (includeTemplate (joinPath .host.distro.id "common-dependencies") .) -}}
{{- $packages = concat $packages $additionalPackages -}}

if [ '{{ .host.distro.id }}' = 'archlinux' ]; then
    ### Print dependency list
    logg 'Installing common dependencies using `pacman`'
    logg info 'Dependencies: {{ $packages | sortAlpha | uniq | join " " -}}'
    
    ### Install packages if they are not already present
    for PACKAGE in {{ $packages | sortAlpha | uniq | join " " -}}; do
        logg info 'Checking for presence of `'"$PACKAGE"'`'
        if pacman -Qs "$PACKAGE" > /dev/null; then
            logg info 'The '"$PACKAGE"' package is already installed'
        else
            logg info 'Installing `'"$PACKAGE"'`'
            sudo pacman -Sy --noconfirm --needed "$PACKAGE" || EXIT_CODE=$?
            if [ -n "$EXIT_CODE" ]; then
                logg error 'Error installing `'"$PACKAGE"'` via pacman'
                logg info 'Proceeding with installation..'
                unset EXIT_CODE
            fi
        fi
    done

    ### Install yay
    if ! command -v yay > /dev/null; then
    logg info 'Cloning yay from `https://aur.archlinux.org/yay.git` to `/usr/local/src/yay`'
    sudo git clone https://aur.archlinux.org/yay.git /usr/local/src/yay
    cd /usr/local/src/yay
    logg info 'Installing yay via `sudo makepkg -si`'
    sudo makepkg -si
    fi
elif [ '{{ .host.distro.id }}' = 'centos' ]; then
    ### Upgrade system
    logg info 'Upgrade system'
    sudo dnf upgrade --refresh -y

    ### Enable CRB
    logg info 'Ensure the CRB repository is activated'
    sudo dnf config-manager --set-enabled crb

    ### Add EPEL
    if ! dnf repolist | grep 'epel ' > /dev/null; then
        logg info 'Adding the EPEL repository'
        sudo dnf install -y "https://dl.fedoraproject.org/pub/epel/epel-release-latest-${VERSION}.noarch.rpm"
    fi

    ### Add EPEL Next
    if ! dnf repolist | grep 'epel-next' > /dev/null; then
        logg info 'Adding the EPEL Next repository'
        sudo dnf install -y "https://dl.fedoraproject.org/pub/epel/epel-next-release-latest-${VERSION}.noarch.rpm"
    else
        logg info 'EPEL Next repository already enabled (EPEL compatibility for CentOS)'
    fi
    ### Detect package manager
    if command -v dnf > /dev/null; then
        PKG_MANAGER='dnf'
    else
        PKG_MANAGER='yum'
    fi

    ### Print dependency list
    logg 'Installing common dependencies using `'"$PKG_MANAGER"'`'
    logg info 'Dependencies: {{ $packages | sortAlpha | uniq | join " " -}}'

    ### Install packages if they are not already present
    for PACKAGE in {{ $packages | sortAlpha | uniq | join " " -}}; do
        logg info 'Checking for presence of `'"$PACKAGE"'`'
        if rpm -qa | grep "^$PACKAGE-" > /dev/null; then
            logg info 'The '"$PACKAGE"' package is already installed'
        else
            logg info 'Installing `'"$PACKAGE"'`'
            sudo "$PKG_MANAGER" install -y "$PACKAGE" || EXIT_CODE=$?
            if [ -n "$EXIT_CODE" ]; then
                logg error 'Error installing `'"$PACKAGE"'` via `'"$PKG_MANAGER"'`'
                logg info 'Proceeding with installation..'
                unset EXIT_CODE
            fi
        fi
    done
elif [ '{{ .host.distro.id }}' = 'debian' ]; then
    ### Print dependency list
    logg 'Installing common dependencies using `apt-get`'
    logg info 'Dependencies: {{ $packages | sortAlpha | uniq | join " " -}}'

    ### Update apt-get cache
    logg info 'Running `sudo apt-get update`'
    sudo apt-get update

    ### Update debconf for non-interactive installation
    if command -v dpkg-reconfigure > /dev/null; then
        logg info 'Running sudo dpkg-reconfigure debconf -f noninteractive -p critical'
        sudo dpkg-reconfigure debconf -f noninteractive -p critical
    fi

    ### Install packages if they are not already present
    for PACKAGE in {{ $packages | sortAlpha | uniq | join " " -}}; do
        logg info 'Checking for presence of `'"$PACKAGE"'`'
        if dpkg -l "$PACKAGE" | grep -E '^ii' > /dev/null; then
            logg info 'The '"$PACKAGE"' package is already installed'
        else
            logg info 'Installing `'"$PACKAGE"'`'
            sudo apt-get install -y --no-install-recommends "$PACKAGE" || EXIT_CODE=$?
            if [ -n "$EXIT_CODE" ]; then
                logg error 'Error installing `'"$PACKAGE"'` via apt-get'
                logg info 'Proceeding with installation..'
                unset EXIT_CODE
            fi
        fi
    done
elif [ '{{ .host.distro.id }}' = 'fedora' ]; then
    ### Upgrade system
    logg info 'Upgrade system'
    sudo dnf upgrade --refresh -y

    # https://docs.fedoraproject.org/en-US/quick-docs/dnf-system-upgrade/
    # TODO - Optional: Look into using Fedora's upgrade system described in the link above
    # sudo dnf install dnf-plugin-system-upgrade

    ### Add RPM Fusion Free repository
    if ! dnf repolist | grep 'rpmfusion-free' > /dev/null; then
        logg info 'Adding RPM-Fusion Free repository for Fedora'
        sudo dnf install -y "https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm"
    fi

    ### Add RPM Fusion Non-Free repository
    if ! dnf repolist | grep 'rpmfusion-nonfree' > /dev/null; then
        logg info 'Adding RPM-Fusion Non-Free repository for Fedora'
        sudo dnf install -y "https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm"
    fi

    ### Add Appstream data from the RPM Fusion repositories
    if command -v gnome-shell > /dev/null; then
        logg info 'Adding Appstream data from the RPM-Fusion repositories'
        sudo dnf group update -y core
    else
        logg warn 'Skipping installation of Appstream data because GNOME is not installed'
    fi
    ### Print dependency list
    logg 'Installing common dependencies using `dnf`'
    logg info 'Dependencies: {{ $packages | sortAlpha | uniq | join " " -}}'

    ### Install packages if they are not already present
    for PACKAGE in {{ $packages | sortAlpha | uniq | join " " -}}; do
        logg info 'Checking for presence of `'"$PACKAGE"'`'
        if rpm -qa | grep "^$PACKAGE-" > /dev/null; then
            logg info 'The '"$PACKAGE"' package is already installed'
        else
            logg info 'Installing `'"$PACKAGE"'`'
            sudo dnf install -y "$PACKAGE" || EXIT_CODE=$?
            if [ -n "$EXIT_CODE" ]; then
                logg error 'Error installing `'"$PACKAGE"'` via dnf'
                logg info 'Proceeding with installation..'
                unset EXIT_CODE
            fi
        fi
    done
elif [ '{{ .host.distro.id }}' = 'freebsd' ]; then
    ### Print dependency list
    logg 'Installing common dependencies using `pkg`'
    logg info 'Dependencies: {{ $packages | sortAlpha | uniq | join " " -}}'

    ### Install base dependencies
    for PACKAGE in {{ $packages | sortAlpha | uniq | join " " -}}; do
        logg info 'Installing `'"$PACKAGE"'`'
        sudo pkg install -y "$PACKAGE" || EXIT_CODE=$?
        if [ -n "$EXIT_CODE" ]; then
            logg error 'Error installing `'"$PACKAGE"'` via zypper'
            logg info 'Proceeding with installation..'
            unset EXIT_CODE
        fi
    done
elif [ '{{ .host.distro.id }}' = 'opensuse' ]; then
    ### Print dependency list
    logg 'Installing common dependencies using `zypper`'
    logg info 'Dependencies: {{ $packages | sortAlpha | uniq | join " " -}}'

    ### Install base_devel
    logg info 'Installing base_devel pattern with `sudo zypper install -t pattern devel_basis`'
    sudo zypper install -t pattern devel_basis

    ### Install packages if they are not already present
    for PACKAGE in {{ $packages | sortAlpha | uniq | join " " -}}; do
        logg info 'Checking for presence of `'"$PACKAGE"'`'
        if rpm -qa | grep "$PACKAGE" > /dev/null; then
            logg info 'The '"$PACKAGE"' package is already installed'
        else
            logg info 'Installing `'"$PACKAGE"'`'
            sudo zypper install -y "$PACKAGE" || EXIT_CODE=$?
            if [ -n "$EXIT_CODE" ]; then
                logg error 'Error installing `'"$PACKAGE"'` via zypper'
                logg info 'Proceeding with installation..'
                unset EXIT_CODE
            fi
        fi
    done
elif [ '{{ .host.distro.id }}' = 'ubuntu' ]; then
    ### Print dependency list
    logg 'Installing common dependencies using `apt-get`'
    logg info 'Dependencies: {{ $packages | sortAlpha | uniq | join " " -}}'

    ### Update apt-get cache
    logg info 'Running `sudo apt-get update`'
    sudo apt-get update

    ### Update debconf for non-interactive installation
    if command -v dpkg-reconfigure > /dev/null; then
        logg info 'Running sudo dpkg-reconfigure debconf -f noninteractive -p critical'
        sudo dpkg-reconfigure debconf -f noninteractive -p critical
    fi

    ### Install packages if they are not already present
    for PACKAGE in {{ $packages | sortAlpha | uniq | join " " -}}; do
        logg info 'Checking for presence of `'"$PACKAGE"'`'
        if dpkg -l "$PACKAGE" | grep -E '^ii' > /dev/null; then
            logg info 'The '"$PACKAGE"' package is already installed'
        else
            logg info 'Installing `'"$PACKAGE"'`'
            sudo apt-get install -y --no-install-recommends "$PACKAGE" || EXIT_CODE=$?
            if [ -n "$EXIT_CODE" ]; then
                logg error 'Error installing `'"$PACKAGE"'` via apt-get'
                logg info 'Proceeding with installation..'
                unset EXIT_CODE
            fi
        fi
    done
fi
{{ end -}}
```
