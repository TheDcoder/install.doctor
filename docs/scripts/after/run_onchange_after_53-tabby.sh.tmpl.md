---
title: Tabby Plugins
description: This script installs the default Tabby plugins which are defined in `${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/package.json`
sidebar_label: 53 Tabby Plugins
slug: /scripts/after/run_onchange_after_53-tabby.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_53-tabby.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_53-tabby.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_53-tabby.sh.tmpl
---
# Tabby Plugins

This script installs the default Tabby plugins which are defined in `${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/package.json`

## Overview

This script pre-installs a handful of useful Tabby plugins which are defined in `${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/package.json`.
These default plugins can be customized by editting the `package.json` file stored in your Install Doctor fork in the Tabby `plugins/package.json`
file.

## Default Plugins Configuration

The script will install all the plugins defined in the `package.json` file by navigating to the `~/.config/tabby/plugins` folder
and then run `npm install`. The default configuration will include the following plugins:

```json
<!-- AUTO-GENERATED:START (REMOTE:url=https://gitlab.com/megabyte-labs/install.doctor/-/raw/master/home/dot_config/tabby/plugins/package.json) -->
{
...
// Notable dependencies listed below
"dependencies": {
"tabby-docker": "^0.2.0",
"tabby-save-output": "^3.1.0",
"tabby-search-in-browser": "^0.0.1",
"tabby-workspace-manager": "^0.0.4"
},
...
}
<!-- AUTO-GENERATED:END -->
```

## Default Plugin Descriptions

The following chart provides a short description of the default plugins that are pre-installed alongside Tabby:

| NPM Package               | Description                                                         |
|---------------------------|---------------------------------------------------------------------|
| `tabby-docker`            | Allows you to shell directly into Docker containers                 |
| `tabby-save-output`       | This plugin lets you stream console output into a file.             |
| `tabby-search-in-browser` | Allows you to open a internet browser and search for selected text. |
| `tabby-workspace-manager` | Allows you to create multiple workspace profiles.                   |

## Links

* [Tabby plugins `package.json`](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/tabby/plugins/package.json)
* [Secrets / Environment variables documentation](https://install.doctor/docs/customization/secrets) which details how to store your Tabby configuration in as an encrypted file



## Source Code

```
{{- if ne .host.distro.family "windows" -}}
#!/usr/bin/env bash
# @file Tabby Plugins
# @brief This script installs the default Tabby plugins which are defined in `${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/package.json`
# @description
#     This script pre-installs a handful of useful Tabby plugins which are defined in `${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/package.json`.
#     These default plugins can be customized by editting the `package.json` file stored in your Install Doctor fork in the Tabby `plugins/package.json`
#     file.
#
#     ## Default Plugins Configuration
#
#     The script will install all the plugins defined in the `package.json` file by navigating to the `~/.config/tabby/plugins` folder
#     and then run `npm install`. The default configuration will include the following plugins:
#
#     ```json
#     <!-- AUTO-GENERATED:START (REMOTE:url=https://gitlab.com/megabyte-labs/install.doctor/-/raw/master/home/dot_config/tabby/plugins/package.json) -->
#     {
#       ...
#       // Notable dependencies listed below
#       "dependencies": {
#         "tabby-docker": "^0.2.0",
#         "tabby-save-output": "^3.1.0",
#         "tabby-search-in-browser": "^0.0.1",
#         "tabby-workspace-manager": "^0.0.4"
#       },
#       ...
#     }
#     <!-- AUTO-GENERATED:END -->
#     ```
#
#     ## Default Plugin Descriptions
#
#     The following chart provides a short description of the default plugins that are pre-installed alongside Tabby:
#
#     | NPM Package               | Description                                                         |
#     |---------------------------|---------------------------------------------------------------------|
#     | `tabby-docker`            | Allows you to shell directly into Docker containers                 |
#     | `tabby-save-output`       | This plugin lets you stream console output into a file.             |
#     | `tabby-search-in-browser` | Allows you to open a internet browser and search for selected text. |
#     | `tabby-workspace-manager` | Allows you to create multiple workspace profiles.                   |
#
#     ## Links
#
#     * [Tabby plugins `package.json`](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_config/tabby/plugins/package.json)
#     * [Secrets / Environment variables documentation](https://install.doctor/docs/customization/secrets) which details how to store your Tabby configuration in as an encrypted file

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

if [ -f "${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/package.json" ]; then
    if [ -d "${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/node_modules" ]; then
        logg info 'Skipping Tabby plugin installation because it looks like the plugins were already installed since `node_modules` is present in ~/.config/tabby/plugins'
    else
        logg info 'Installing Tabby plugins defined in `'"${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins/package.json"'`'
        cd "${XDG_CONFIG_HOME:-$HOME/.config}/tabby/plugins"
        npm install
        logg success 'Finished installing Tabby plugins'
    fi
else
    logg info 'Skipping Tabby plugin installation because is not present'
fi

{{ end -}}
```
