---
title: Node.js Version
description: Ensures a recent version of Node.js is available to the user by leveraging Volta
sidebar_label: 09 Node.js Version
slug: /scripts/before/run_onchange_before_09-node-version.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_09-node-version.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_09-node-version.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_09-node-version.sh.tmpl
---
# Node.js Version

Ensures a recent version of Node.js is available to the user by leveraging Volta

## Overview

This script installs the latest version of Node.js with Volta if the default Node.js version
is an outdated version.



## Source Code

```
{{- if (ne .host.distro.family "windows") -}}
#!/usr/bin/env bash
# @file Node.js Version
# @brief Ensures a recent version of Node.js is available to the user by leveraging Volta
# @description
#     This script installs the latest version of Node.js with Volta if the default Node.js version
#     is an outdated version.

# Node.js version: {{ output "node" "--version" }}

{{ includeTemplate "universal/profile-before" }}
{{ includeTemplate "universal/logg-before" }}

### Ensure recent version of Node.js is being used
if command -v volta > /dev/null; then
  if ! test "$(node --version | sed 's/^v//' | awk '{print $1}' | awk -F'.' ' ( $1 > 15) ')"; then
    logg info 'Installing latest version of Node.js'
    volta install node@latest
  else
    logg info 'Node.js appears to meet the minimum version requirements (version >15)'
  fi
else
  logg warn 'Volta is not installed - skipping logic that ensures Node.js meets the version requirement of >15'
fi

{{ end -}}
```
