---
title: Qubes Update dom0
description: Ensures Qubes dom0 is up-to-date, includes all the Qubes repository definitions, and that `sys-whonix` is running
sidebar_label: 12 Qubes Update dom0
slug: /scripts/before/run_onchange_before_12-update-dom0.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_12-update-dom0.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_12-update-dom0.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_12-update-dom0.sh.tmpl
---
# Qubes Update dom0

Ensures Qubes dom0 is up-to-date, includes all the Qubes repository definitions, and that `sys-whonix` is running

## Overview

This script begins with ensuring Qubes dom0 is up-to-date by including developer-channel Qubes repository definitions are available.
It then ensures Qubes dom0 is up-to-date using a combination of several methods including the recommended Salt-based
update script. After dom0 is updated, it installs the packages defined in `.qubes.dom0Packages` which is defined in the
`home/.chezmoidata.yaml` file. It also ensures `sys-whonix` is running if it is not already running.



## Source Code

```
{{- if (eq .host.distro.id "qubes") -}}
#!/usr/bin/env bash
# @file Qubes Update dom0
# @brief Ensures Qubes dom0 is up-to-date, includes all the Qubes repository definitions, and that `sys-whonix` is running
# @description
#     This script begins with ensuring Qubes dom0 is up-to-date by including developer-channel Qubes repository definitions are available.
#     It then ensures Qubes dom0 is up-to-date using a combination of several methods including the recommended Salt-based
#     update script. After dom0 is updated, it installs the packages defined in `.qubes.dom0Packages` which is defined in the
#     `home/.chezmoidata.yaml` file. It also ensures `sys-whonix` is running if it is not already running.

# qubes-templates.repo hash: {{ include (joinPath .chezmoi.homeDir ".config" "qubes" "qubes-templates.repo") | sha256sum }}
# qubes-dom0.repo hash: {{ include (joinPath .chezmoi.homeDir ".config" "qubes" "qubes-dom0.repo") | sha256sum }}
# qubes packages: {{ .qubes.dom0Packages | toString | replace "[" "" | replace "]" "" }}

### Configure dom0 repos
logg info 'Updating dom0 repos to include auxilary branches'
sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/qubes/qubes-templates.repo" /etc/qubes/repo-templates/qubes-templates.repo
sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/qubes/qubes-dom0.repo" /etc/yum.repos.d/qubes-dom0.repo

### Update dom0
logg info 'Updating dom0 via `qubesctl`'
sudo qubesctl --show-output state.sls update.qubes-dom0
logg info 'Updating dom0 via `qubes-dom0-update`'
sudo qubes-dom0-update --clean -y

### Install qubes-repo-contrib
logg info "Installing qubes-repo-contrib"
sudo qubes-dom0-update -y qubes-repo-contrib

### Install dependencies
for PACKAGE of {{ .qubes.dom0Packages | toString | replace "[" "" | replace "]" "" }}; do
  logg info "Installing $PACKAGE"
  sudo qubes-dom0-update -y "$PACKAGE" || true
done

### Ensure sys-whonix is running
logg info 'Ensuring `sys-whonix` is running'
qvm-start sys-whonix --skip-if-running
{{ end -}}
```
