---
title: Linux Swap
description: Determines the ideal size `/swapfile`, ensures it exists, and then enables it on Linux systems
sidebar_label: 11 Linux Swap
slug: /scripts/before/run_onchange_before_11-configure-swap.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_11-configure-swap.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_11-configure-swap.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_11-configure-swap.sh.tmpl
---
# Linux Swap

Determines the ideal size `/swapfile`, ensures it exists, and then enables it on Linux systems

## Overview

This script determines the ideal `/swapfile` size by checking how much RAM is available on the system.
It then creates the appropriate `/swapfile` by considering factors such as the file system type. It
currently supports BTRFS and regular file systems.

After the `/swapfile` is created, it is enabled and assigned the appropriate permissions.

## TODO

* Add logic that creates a swapfile for ZFS-based systems
* Integrate logic from https://gitlab.com/megabyte-labs/gas-station/-/blob/master/roles/system/common/tasks/linux/swap.yml



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file Linux Swap
# @brief Determines the ideal size `/swapfile`, ensures it exists, and then enables it on Linux systems
# @description
#     This script determines the ideal `/swapfile` size by checking how much RAM is available on the system.
#     It then creates the appropriate `/swapfile` by considering factors such as the file system type. It
#     currently supports BTRFS and regular file systems.
#
#     After the `/swapfile` is created, it is enabled and assigned the appropriate permissions.
#
#     ## TODO
#
#     * Add logic that creates a swapfile for ZFS-based systems
#     * Integrate logic from https://gitlab.com/megabyte-labs/gas-station/-/blob/master/roles/system/common/tasks/linux/swap.yml

{{ includeTemplate "universal/profile-before" }}
{{ includeTemplate "universal/logg-before" }}

if [ ! -f /swapfile ]; then
  ### Determine ideal size of /swapfile
  MEMORY_IN_KB="$(grep MemTotal /proc/meminfo | sed 's/.* \(.*\) kB/\1/')"
  MEMORY_IN_GB="$((MEMORY_IN_KB / 1024 / 1024))"
  if [ "$MEMORY_IN_GB" -gt 64 ]; then
    SWAP_SPACE="$((MEMORY_IN_GB / 10))"
  elif [ "$MEMORY_IN_GB" -gt 32 ]; then
    SWAP_SPACE="$((MEMORY_IN_GB / 8))"
  elif [ "$MEMORY_IN_GB" -gt 8 ]; then
    SWAP_SPACE="$((MEMORY_IN_GB / 4))"
  else
    SWAP_SPACE="$MEMORY_IN_GB"
  fi
  
  ### Create /swapfile
  FS_TYPE="$(df -Th | grep ' /$' | sed 's/[^ ]*\s*\([^ ]*\).*/\1/')"
  if [ "$FS_TYPE" == 'btrfs' ]; then
    logg info 'Creating BTRFS /swapfile'
    sudo btrfs filesystem mkswapfile /swapfile
  elif [ "$FS_TYPE" == 'zfs' ]; then
    logg warn 'ZFS system detected - add logic here to add /swapfile'
  else
    logg info "Creating a $SWAP_SPACE GB /swapfile"
    sudo fallocate -l "${SWAP_SPACE}G" /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
  fi

  ### Enable the /swapfile
  if [ -f /swapfile ]; then
    logg info 'Running `sudo swapon /swapfile`'
    sudo swapon /swapfile
    if cat /etc/fstab | grep "/swapfile"; then
      sudo sed -i '/\/swapfile/\/swapfile none swap defaults 0 0/' /etc/fstab
    else
      echo "/swapfile none swap defaults 0 0" | sudo tee -a /etc/fstab > /dev/null
    fi
  fi
fi
{{ end -}}
```
