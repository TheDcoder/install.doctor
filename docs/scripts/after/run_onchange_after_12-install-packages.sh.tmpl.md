---
title: Software Installation
description: Installs the list of software that correlates to the software group that was chosen.
sidebar_label: 12 Software Installation
slug: /scripts/after/run_onchange_after_12-install-packages.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_12-install-packages.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_12-install-packages.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_12-install-packages.sh.tmpl
---
# Software Installation

Installs the list of software that correlates to the software group that was chosen.

## Overview

This script initializes the installation process that handles the bulk of the software package installations.



## Source Code

```
#!/usr/bin/env bash
# @file Software Installation
# @brief Installs the list of software that correlates to the software group that was chosen.
# @description
#     This script initializes the installation process that handles the bulk of the software package installations.

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}
{{- $softwareGroup := nospace (cat "_" .host.softwareGroup) }}
{{- $softwareList := list (index .softwareGroups $softwareGroup | toString | replace "[" "" | replace "]" "") | uniq | join " " }}

# software: {{ $softwareList }}
# software map: {{ include (joinPath .chezmoi.homeDir ".local" "share" "chezmoi" "software.yml") | sha256sum }}

if command -v install-program > /dev/null; then
  if command -v zx > /dev/null; then
    logg info 'Installing packages defined in .chezmoidata.yaml under the .softwareGroups key'
    logg info 'Installing: {{ $softwareList }}'
    # Ask for the administrator password upfront
    logg info 'A sudo password may be required for some of the installations'
    sudo echo "Sudo access granted."
    export DEBIAN_FRONTEND=noninteractive
    export HOMEBREW_NO_ENV_HINTS=true
    if ! command -v gcc-11; then
      if command -v gcc; then
        log info 'gcc-11 command missing. Symlinking to gcc'
        sudo ln -s "$(which gcc)" /usr/local/bin/gcc-11
      else
        log warn 'gcc either needs to be added to the PATH or it is missing'
      fi
    fi
    if [ -f "$HOME/.bashrc" ]; then
      . "$HOME/.bashrc"
    else
      logg warn 'No ~/.bashrc file to import before running install-program'
    fi
    export LC_ALL="en_US.UTF-8"
    logg info 'Printing environment variables for GO'
    env | grep GO
    install-program {{ $softwareList }}
    # TODO - Figure out how to configure no logs to print to ~/.ansible.log -- should be printing to the value specified in the ansible.cfg
    rm -rf "$HOME/.ansible.log"
  else
    logg error '`zx` is not available'
  fi
else
  logg error '`install-program` is not in the PATH. It should be located in ~/.local/bin.'
fi
```
