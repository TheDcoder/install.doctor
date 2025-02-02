---
title: Ansible Role Symlinks
description: Symlinks roles that are part of the [Gas Station](https://github.com/megabyte-labs/gas-station) project to a location that Ansible can digest
sidebar_label: 11 Ansible Role Symlinks
slug: /scripts/after/run_onchange_after_11-symlink-ansible-roles.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_11-symlink-ansible-roles.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_11-symlink-ansible-roles.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_11-symlink-ansible-roles.sh.tmpl
---
# Ansible Role Symlinks

Symlinks roles that are part of the [Gas Station](https://github.com/megabyte-labs/gas-station) project to a location that Ansible can digest

## Overview

Install Doctor was previously called Gas Station. It was also Ansible based. Some of the features that Install Doctor
provides are made available via Ansible roles that Gas Station provides. This script symlinks Gas Station's roles
so that they can be leveraged by Install Doctor.

Some of the roles that Gas Station provides are not available via Ansible Galaxy yet. This script symlinks Gas Station
roles to an Ansible Galaxy / Ansible friendly location.

## Ansible Installation

If Ansible is not already installed, this script will also install Ansible and all the necessary requirements using `pipx`.
This script must run before the `install-packages` script because some of the Ansible roles might be leveraged by it.

## TODO

* Move installation logic into the ZX installer so that Ansible and its dependencies are only installed when required
* Remove Ansible dependency completely



## Source Code

```
#!/usr/bin/env bash
# @file Ansible Role Symlinks
# @brief Symlinks roles that are part of the [Gas Station](https://github.com/megabyte-labs/gas-station) project to a location that Ansible can digest
# @description
#     Install Doctor was previously called Gas Station. It was also Ansible based. Some of the features that Install Doctor
#     provides are made available via Ansible roles that Gas Station provides. This script symlinks Gas Station's roles
#     so that they can be leveraged by Install Doctor.
#
#     Some of the roles that Gas Station provides are not available via Ansible Galaxy yet. This script symlinks Gas Station
#     roles to an Ansible Galaxy / Ansible friendly location.
#
#     ## Ansible Installation
#
#     If Ansible is not already installed, this script will also install Ansible and all the necessary requirements using `pipx`.
#     This script must run before the `install-packages` script because some of the Ansible roles might be leveraged by it.
#
#     ## TODO
#
#     * Move installation logic into the ZX installer so that Ansible and its dependencies are only installed when required
#     * Remove Ansible dependency completely

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

{{ $roleDirs := (output "find" (joinPath .chezmoi.homeDir ".local" "src" "gas-station" "roles") "-mindepth" "2" "-maxdepth" "2" "-type" "d") -}}
{{- range $roleDir := splitList "\n" $roleDirs -}}
{{- if ne $roleDir "" -}}
# {{ $roleDir }}
{{- end -}}
{{- end }}

logg info 'Ensuring Gas Station roles are symlinked to ~/.local/share/ansible/roles'
mkdir -p "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles"
find "$HOME/.local/src/gas-station/roles" -mindepth 2 -maxdepth 2 -type d | while read ROLE_PATH; do
  ROLE_FOLDER="professormanhattan.$(echo "$ROLE_PATH" | sed 's/.*\/\([^\/]*\)$/\1/')"
  ALT_ROLE_FOLDER="$(echo "$ROLE_PATH" | sed 's/.*\/\([^\/]*\)$/\1/')"
  if [ ! -d "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ROLE_FOLDER" ] || [ "$(readlink -f "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ROLE_FOLDER")" != "$ROLE_PATH" ]; then
    logg info 'Symlinking `'"$ROLE_FOLDER"'`'
    rm -f "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ROLE_FOLDER"
    ln -s "$ROLE_PATH" "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ROLE_FOLDER"
  fi
  if [ ! -d "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ALT_ROLE_FOLDER" ] || [ "$(readlink -f "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ALT_ROLE_FOLDER")" != "$ROLE_PATH" ]; then
    rm -f "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ALT_ROLE_FOLDER"
    ln -s "$ROLE_PATH" "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/roles/$ALT_ROLE_FOLDER"
  fi
done

if [ -f "$HOME/.local/src/gas-station/requirements.yml" ]; then
  ### Install Ansible Galaxy and dependencies if missing
  if ! command -v ansible-galaxy > /dev/null; then
    if ! command -v pipx > /dev/null; then
      logg info 'Installing pipx via Homebrew'
      brew install pipx
      logg info 'Running `pipx ensurepath`'
      pipx ensurepath
    fi
    logg info 'Installing `ansible-core` via pipx'
    pipx install ansible-core
    if [ -d /Applications ] && [ -d /System ]; then
      logg info 'Injecting macOS-specific pipx dependencies via pipx'
      pipx inject ansible-core PyObjC PyObjC-core
    fi
    logg info 'Injecting Ansible dependencies via pipx'
    pipx inject ansible-core docker lxml netaddr pexpect python-vagrant pywinrm requests-credssp watchdog
    mkdir -p "${XDG_CACHE_HOME:-$HOME/.cache}/megabyte-labs"
    touch "${XDG_CACHE_HOME:-$HOME/.cache}/megabyte-labs/ansible-installed"
  fi

  ### Ensure Ansible Galaxy was successfully loaded and then install the Ansible Galaxy requirements
  if command -v ansible-galaxy > /dev/null; then
    logg info 'Ensuring Ansible Galaxy collections are installed'
    export ANSIBLE_CONFIG="${XDG_DATA_HOME:-$HOME/.local/share}/ansible/ansible.cfg"
    ansible-galaxy install -r "${XDG_DATA_HOME:-$HOME/.local/share}/ansible/requirements.yml" || EXIT_CODE=$?
    if [ -n "$EXIT_CODE" ]; then
      logg error 'Failed to install Ansible requirements from Ansible Galaxy'
      if [ -d "$HOME/.local/src/gas-station/collections" ]; then
        logg info 'Attempting to use locally stored Ansible requirements'
        cd "$HOME/.local/src/gas-station/collections"
        ansible-galaxy install -r requirements.yml || SECOND_EXIT_CODE=$?
        if [ -n "$SECOND_EXIT_CODE" ]; then
          logg error 'Failed to install requirements from both the cloud and the local copy' && exit 1
        fi
      else
        logg warn '~/.local/src/gas-station/collections is missing'
      fi
    fi
  else
    logg warn 'Unable to install the Ansible Galaxy requirements.yml since the ansible-galaxy executable is missing from the PATH'
  fi
else
  logg warn '~/.local/share/ansible/requirements.yml is missing'
fi
```
