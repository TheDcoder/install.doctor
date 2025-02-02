---
title: VMWare Configuration
description: Patches VMWare to leverage kernel optimizations, support macOS, and work harmoniously with Secure Boot. It also enables optional services such as the USB service.
sidebar_label: 45 VMWare Configuration
slug: /scripts/after/run_onchange_after_45-vmware.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_45-vmware.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_45-vmware.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_45-vmware.sh.tmpl
---
# VMWare Configuration

Patches VMWare to leverage kernel optimizations, support macOS, and work harmoniously with Secure Boot. It also enables optional services such as the USB service.

## Overview

This script performs various VMWare optimizations that allow VMWare to work optimally with all features enabled.



## Source Code

```
{{- if eq .host.distro.family "linux" -}}
#!/usr/bin/env bash
# @file VMWare Configuration
# @brief Patches VMWare to leverage kernel optimizations, support macOS, and work harmoniously with Secure Boot. It also enables optional services such as the USB service.
# @description
#     This script performs various VMWare optimizations that allow VMWare to work optimally with all features enabled.

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Run logic if VMware is installed
if command -v vmware > /dev/null; then
  ### Build kernel modules if they are not present
  if [ ! -f "/lib/modules/$(uname -r)/misc/vmmon.ko" ] || [ ! -f "/lib/modules/$(uname -r)/misc/vmnet.ko" ]; then
    ### Build VMWare host modules
    logg info 'Building VMware host modules'
    if sudo vmware-modconfig --console --install-all; then
      logg success 'Built VMWare host modules successfully with `sudo vmware-modconfig --console --install-all`'
    else
      logg info 'Acquiring VMware version from CLI'
      VMW_VERSION="$(vmware --version | cut -f 3 -d' ')"
      mkdir -p /tmp/vmw_patch
      cd /tmp/vmw_patch
      logg info 'Downloading VMware host module patches'
      curl -sSL "https://github.com/mkubecek/vmware-host-modules/archive/workstation-$VMW_VERSION.tar.gz" -o /tmp/vmw_patch/workstation.tar.gz
      tar -xzf /tmp/vmw_patch/workstation.tar.gz
      cd vmware*
      logg info 'Running `sudo make` and `sudo make install`'
      sudo make
      sudo make install
      logg success 'Successfully configured VMware host module patches'
    fi

    ### Sign VMware host modules if Secure Boot is enabled
    if [ -f /sys/firmware/efi ]; then
      logg info 'Signing host modules'
      mkdir -p /tmp/vmware
      cd /tmp/vmware
      openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=VMware/"
      "/usr/src/linux-headers-$(uname -r)/scripts/sign-file" sha256 ./MOK.priv ./MOK.der "$(modinfo -n vmmon)"
      "/usr/src/linux-headers-$(uname -r)/scripts/sign-file" sha256 ./MOK.priv ./MOK.der "$(modinfo -n vmnet)"
      echo '' | mokutil --import MOK.der
      logg success 'Successfully signed VMware host modules. Reboot the host before powering on VMs'
    fi

    ### Patch VMware with Unlocker
    if [ ! -f /usr/lib/vmware/isoimages/darwin.iso ]; then
      logg info 'Acquiring VMware Unlocker latest release version'
      UNLOCKER_URL="$(curl -sSL 'https://api.github.com/repos/DrDonk/unlocker/releases/latest' | jq  -r '.assets[0].browser_download_url')"
      mkdir -p /tmp/vmware-unlocker
      cd /tmp/vmware-unlocker
      logg info 'Downloading unlocker.zip'
      curl -sSL "$UNLOCKER_URL" -o unlocker.zip
      unzip unlocker.zip
      cd linux
      logg info 'Running the unlocker'
      echo "y" | sudo ./unlock
      logg success 'Successfully unlocked VMware for macOS compatibility'
    else
      logg info '/usr/lib/vmware/isoimages/darwin.iso is already present on the system so VMware macOS unlocking will not be performed'
    fi

    if [[ ! "$(test -d /proc && grep Microsoft /proc/version > /dev/null)" ]]; then
      ### Start / enable VMWare service
      logg info 'Ensuring `vmware.service` is enabled and running'
      sudo systemctl enable vmware.service
      sudo systemctl restart vmware.service

      ### Start / enable VMWare Workstation Server service
      logg info 'Ensuring `vmware-workstation-server.service` is enabled and running'
      sudo systemctl enable vmware-workstation-server.service
      sudo systemctl restart vmware-workstation-server.service

      ### Start / enable VMWare USB Arbitrator service
      if command -v vmware-usbarbitrator.service > /dev/null; then
        logg info 'Ensuring `vmware-usbarbitrator.service` is enabled and running'
        sudo systemctl enable vmware-usbarbitrator.service
        sudo systemctl restart vmware-usbarbitrator.service
      else
        logg warn '`vmware-usbarbitrator` does not exist in the PATH'
      fi
    fi
  else
    logg info 'VMware host modules are present'
  fi
else
  logg warn 'VMware Workstation is not installed so the VMware Unlocker will not be installed'
fi

{{ end -}}
```
