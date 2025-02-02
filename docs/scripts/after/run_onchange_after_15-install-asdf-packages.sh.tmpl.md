---
title: ASDF Plugins / Install
description: Configures ASDF plugins and ensures they are pre-installed.
sidebar_label: 15 ASDF Plugins / Install
slug: /scripts/after/run_onchange_after_15-install-asdf-packages.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_15-install-asdf-packages.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_15-install-asdf-packages.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_15-install-asdf-packages.sh.tmpl
---
# ASDF Plugins / Install

Configures ASDF plugins and ensures they are pre-installed.

## Overview

This script ensures ASDF is setup and then adds the plugins specified in the `~/.tool-versions` file. After that,
it ensures the ASDF plugins are pre-installed.



## Source Code

```
#!/usr/bin/env bash
{{- if (ne .host.distro.family "windows") }}
# @file ASDF Plugins / Install
# @brief Configures ASDF plugins and ensures they are pre-installed.
# @description
#     This script ensures ASDF is setup and then adds the plugins specified in the `~/.tool-versions` file. After that,
#     it ensures the ASDF plugins are pre-installed.

# dot_tool-versions.tmpl hash: {{ include "dot_tool-versions.tmpl" | sha256sum }}

{{- includeTemplate "universal/profile" }}
{{- includeTemplate "universal/logg" }}

if [ -f "$ASDF_DIR/asdf.sh" ] && [ -f ~/.tool-versions ]; then
  logg info 'Sourcing asdf.sh'
  . ${ASDF_DIR}/asdf.sh
  cat .tool-versions | while read TOOL; do
    logg info 'Installing ASDF plugin `'"$(echo "$TOOL" | sed 's/ .*//')"'`'
    asdf plugin add "$(echo "$TOOL" | sed 's/ .*//')"
  done
  # Only proceed with installation if either DEBUG_MODE is enabled or ~/.cache/megabyte-labs/asdf-install is missing
  # Added to save time between tests because PHP takes awhile to install
  if [ "$DEBUG_MODE" == 'true' ] || [ ! -f "${XDG_CACHE_HOME:-$HOME/.cache}/megabyte-labs/asdf-install" ]; then
    logg info 'Installing ASDF dependencies derived from `~/.tool-versions` via `asdf install`'
    asdf install || EXIT_CODE=$?
    if [ -n "$EXIT_CODE" ]; then
      logg error 'Error installing the ASDF plugins specified in `~/.tool-versions`'
    fi
    mkdir -p "${XDG_CACHE_HOME:-$HOME/.cache}/megabyte-labs"
    touch "${XDG_CACHE_HOME:-$HOME/.cache}/megabyte-labs/asdf-install"
  fi
else
  logg warn 'The `$ASDF_DIR/asdf.sh` or `~/.tool-versions` file is not present'
fi

{{ end -}}
```
