---
title: Bash It!
description: Ensures Bash is configured to use Bash It!
sidebar_label: 94 Bash It!
slug: /scripts/after/run_onchange_after_94-bash-it.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_94-bash-it.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_94-bash-it.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_94-bash-it.sh.tmpl
---
# Bash It!

Ensures Bash is configured to use Bash It!

## Overview

This script ensures Bash is configured to use Bash It! It ensures dependencies are installed, installs completions,
and enables Bash It! plugins. The completions and plugins are hardcoded in this script.



## Source Code

```
{{- if (ne .host.distro.family "windows") -}}
#!/usr/bin/env -S bash -i
# @file Bash It!
# @brief Ensures Bash is configured to use Bash It!
# @description
#     This script ensures Bash is configured to use Bash It! It ensures dependencies are installed, installs completions,
#     and enables Bash It! plugins. The completions and plugins are hardcoded in this script.

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Ensure Powerline is installed
if ! command -v powerline > /dev/null; then
    install-software powerline
fi

### Bash-it completions / plugins
if command -v powerline > /dev/null && [ -f "$HOME/.bashrc" ]; then
    logg info 'Running `source ~/.bashrc`'
    source ~/.bashrc
    logg success 'Imported the `~/.bashrc` profile'
    if command -v bash-it > /dev/null; then
        if [ -n "$BASH_IT" ]; then
            cd "$BASH_IT" || logg warn "The $BASH_IT directory does not exist"
            logg info 'Enabling bash-it completions'
            yes | bash-it enable completion defaults dirs docker docker-compose export git makefile ng npm ssh system vagrant
            logg info 'Enabling bash-it plugins'
            yes | bash-it enable plugin base blesh browser cht-sh dirs gitstatus powerline sudo xterm
            logg info 'Finished enabling bash-it functions'
        else
            logg warn 'The BASH_IT variable needs to be defined'
        fi
    else
        logg warn '`bash-it` is not available'
    fi
else
    if ! command -v powerline > /dev/null; then
        logg warn '`powerline` is not available'
    else
        logg warn '`~/.bashrc` is missing'
    fi
fi

{{ end -}}
```
