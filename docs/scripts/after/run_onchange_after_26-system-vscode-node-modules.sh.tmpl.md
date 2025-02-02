---
title: Global NPM Modules Fallback
description: Installs NPM packages to the system `/` directory as a catch-all for tools that recursively search upwards for shared NPM configurations.
sidebar_label: 26 Global NPM Modules Fallback
slug: /scripts/after/run_onchange_after_26-system-vscode-node-modules.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_26-system-vscode-node-modules.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_26-system-vscode-node-modules.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_26-system-vscode-node-modules.sh.tmpl
---
# Global NPM Modules Fallback

Installs NPM packages to the system `/` directory as a catch-all for tools that recursively search upwards for shared NPM configurations.

## Overview

This script makes fallback linter and code auto-fixer configurations globally available. Normally, configurations, like
the ones used for ESLint, are installed at the project level by specifying the NPM package configuration
in the `package.json` file (or via an `.eslintrc` file). However, whenever no configuration is present, IDEs like
Visual Studio Code will recursively search upwards in the directory tree, trying to find an ESLint configuration.

This script addresses this issue by installing a set of shared NPM packages that enhance the functionality of tools like ESLint
by placing a `package.json` with all the necessary settings into the highest directory possible and then installing the package's
modules. This normally results in a `package.json` file and `node_modules/` folder at the root of the system.

## NPM Packages Included

To reduce clutter, all the configurations are mapped out in the `package.json` file. Our default `package.json` file includes
the following configuration:

```json
<!-- AUTO-GENERATED:START (REMOTE:url=https://gitlab.com/megabyte-labs/install.doctor/-/raw/master/home/dot_config/Code/User/package.json) -->
{
...
// Notable dependencies listed below
"dependencies": {
"eslint-config-strictlint": "latest",
"jest-preset-ts": "latest",
"prettier-config-strictlint": "latest",
"remark-preset-strictlint": "latest",
"stylelint-config-strictlint": "latest"
},
...
}
<!-- AUTO-GENERATED:END -->
```

## Strict Lint

More details on the shared configurations can be found at [StrictLint.com](https://strictlint.com).
Strict Lint is another brand maintained by Megabyte Labs that is home to many of the well-crafted
shared configurations that are included in our default NPM configuration fallback settings.

## Notes

* If the system root directory is not writable (even with `sudo`), then the shared modules are installed to the provisioning user's `$HOME` directory

## Links

* [`package.json` configuration file](https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/Code/User/package.json)
* [StrictLint.com documentation](https://strictlint.com/docs)



## Source Code

```
{{- if (ne .host.distro.family "windows") -}}
#!/usr/bin/env bash
# @file Global NPM Modules Fallback
# @brief Installs NPM packages to the system `/` directory as a catch-all for tools that recursively search upwards for shared NPM configurations.
# @description
#     This script makes fallback linter and code auto-fixer configurations globally available. Normally, configurations, like
#     the ones used for ESLint, are installed at the project level by specifying the NPM package configuration
#     in the `package.json` file (or via an `.eslintrc` file). However, whenever no configuration is present, IDEs like
#     Visual Studio Code will recursively search upwards in the directory tree, trying to find an ESLint configuration.
#
#     This script addresses this issue by installing a set of shared NPM packages that enhance the functionality of tools like ESLint
#     by placing a `package.json` with all the necessary settings into the highest directory possible and then installing the package's
#     modules. This normally results in a `package.json` file and `node_modules/` folder at the root of the system.
#
#     ## NPM Packages Included
#
#     To reduce clutter, all the configurations are mapped out in the `package.json` file. Our default `package.json` file includes
#     the following configuration:
#
#     ```json
#     <!-- AUTO-GENERATED:START (REMOTE:url=https://gitlab.com/megabyte-labs/install.doctor/-/raw/master/home/dot_config/Code/User/package.json) -->
#     {
#       ...
#       // Notable dependencies listed below
#       "dependencies": {
#         "eslint-config-strictlint": "latest",
#         "jest-preset-ts": "latest",
#         "prettier-config-strictlint": "latest",
#         "remark-preset-strictlint": "latest",
#         "stylelint-config-strictlint": "latest"
#       },
#       ...
#     }
#     <!-- AUTO-GENERATED:END -->
#     ```
#
#     ## Strict Lint
#
#     More details on the shared configurations can be found at [StrictLint.com](https://strictlint.com).
#     Strict Lint is another brand maintained by Megabyte Labs that is home to many of the well-crafted
#     shared configurations that are included in our default NPM configuration fallback settings.
#
#     ## Notes
#
#     * If the system root directory is not writable (even with `sudo`), then the shared modules are installed to the provisioning user's `$HOME` directory
#
#     ## Links
#
#     * [`package.json` configuration file](https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/Code/User/package.json)
#     * [StrictLint.com documentation](https://strictlint.com/docs)

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

if command -v code > /dev/null && command -v pnpm > /dev/null && [ -f "${XDG_DATA_HOME:-$HOME/.local/share}/vscode/package.json" ]; then
    ### Install linter fallback node_modules / package.json to system or home directory
    if sudo cp -f "${XDG_DATA_HOME:-$HOME/.local/share}/vscode/package.json" /package.json; then
        logg info 'Successfully copied linter fallback configurations package.json to /package.json'
        logg info 'Installing system root directory node_modules'
        cd / && sudo pnpm i --no-lockfile || EXIT_CODE=$?
    else
        logg warn 'Unable to successfully copy linter fallback configurations package.json to /package.json'
        logg info 'Installing linter fallback configurations node_modules to home directory instead'
        cp -f "${XDG_DATA_HOME:-$HOME/.local/share}/vscode/package.json" "$HOME/package.json"
        cd ~ && pnpm i --no-lockfile || EXIT_CODE=$?
    fi

    ### Log message if install failed
    if [ -n "$EXIT_CODE" ]; then
        logg warn 'Possible error(s) were detected while installing linter fallback configurations to the home directory.'
    else
        logg success 'Installed linter fallback configuration node_modules'
    fi
else
    logg info 'Skipping installation of fallback linter configurations because one or more of the dependencies is missing.'
fi

{{ end -}}
```
