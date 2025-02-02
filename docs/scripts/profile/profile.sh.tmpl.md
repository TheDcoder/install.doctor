---
title: Shared Profile
description: Main shell profile that is used to combine the shared profile configurations that are used by both the `~/.bashrc` and `~/.zshrc` files
sidebar_label: Shared Profile
slug: /scripts/profile/profile.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/shell/profile.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/dot_config/shell/profile.sh.tmpl
repoLocation: home/dot_config/shell/profile.sh.tmpl
---
# Shared Profile

Main shell profile that is used to combine the shared profile configurations that are used by both the `~/.bashrc` and `~/.zshrc` files

## Overview

This script is included by `~/.bashrc` and `~/.zshrc` to include imports and settings that are common to both the Bash
and ZSH shells.



## Source Code

```
#!/usr/bin/env sh
# @file Shared Profile
# @brief Main shell profile that is used to combine the shared profile configurations that are used by both the `~/.bashrc` and `~/.zshrc` files
# @description
#     This script is included by `~/.bashrc` and `~/.zshrc` to include imports and settings that are common to both the Bash
#     and ZSH shells.

# shellcheck disable=SC1090,SC1091

# Aliases / Functions / Exports
export XDG_CONFIG_HOME="$HOME/.config"
if [ -f "$XDG_CONFIG_HOME/shell/exports.sh" ]; then
  . "$XDG_CONFIG_HOME/shell/exports.sh"
fi
if [ -f "$XDG_CONFIG_HOME/shell/aliases.sh" ]; then
  . "$XDG_CONFIG_HOME/shell/aliases.sh"
fi
if [ -f "$XDG_CONFIG_HOME/shell/functions.sh" ]; then
  . "$XDG_CONFIG_HOME/shell/functions.sh"
fi

### Bash / ZSH
if [ "$BASH_SUPPORT" = 'true' ]; then
  ### OS Detection
  if [ -f /etc/os-release ]; then
    . /etc/os-release
    if [ "$ID" = 'alpine' ]; then
      OS_ICON=""
    elif [ "$ID" = 'arch' ]; then
      OS_ICON=""
    elif [ "$ID" = 'centos' ]; then
      OS_ICON=""
    elif [ "$ID" = 'coreos' ]; then
      OS_ICON=""
    elif [ "$ID" = 'debian' ]; then
      OS_ICON=""
    elif [ "$ID" = 'deepin' ]; then
      OS_ICON=""
    elif [ "$ID" = 'elementary' ]; then
      OS_ICON=""
    elif [ "$ID" = 'endeavour' ]; then
      OS_ICON=""
    elif [ "$ID" = 'freebsd' ]; then
      OS_ICON=""
    elif [ "$ID" = 'gentoo' ]; then
      OS_ICON=""
    elif [ "$ID" = 'kali' ]; then
      OS_ICON=""
    elif [ "$ID" = 'linuxmint' ]; then
      OS_ICON=""
    elif [ "$ID" = 'manjaro' ]; then
      OS_ICON=""
    elif [ "$ID" = 'nixos' ]; then
      OS_ICON=""
    elif [ "$ID" = 'openbsd' ]; then
      OS_ICON=""
    elif [ "$ID" = 'opensuse' ]; then
      OS_ICON=""
    elif [ "$ID" = 'parrot' ]; then
      OS_ICON=""
    elif [ "$ID" = 'pop_os' ]; then
      OS_ICON=""
    elif [ "$ID" = 'raspberry_pi' ]; then
      OS_ICON=""
    elif [ "$ID" = 'redhat' ]; then
      OS_ICON=""
    elif [ "$ID" = 'fedora' ]; then
      OS_ICON=""
    elif [ "$ID" = 'ubuntu' ]; then
      OS_ICON=""
    else
      OS_ICON=""
    fi
  else
    if [ -d /Applications ] && [ -d /Library ] && [ -d /System ]; then
      # macOS
      OS_ICON=""
    else
      OS_ICON=""
    fi
  fi

  ### ASDF
  if [ -f "$ASDF_DIR/asdf.sh" ]; then
    . "$ASDF_DIR/asdf.sh"
  fi

  ### Directory Colors
  if [ -f "$XDG_CONFIG_HOME/shell/lscolors.sh" ]; then
    . "$XDG_CONFIG_HOME/shell/lscolors.sh"
  fi

  ### fzf-git
  #if [ -f "$HOME/.local/scripts/fzf-git.bash" ]; then
  #  . "$HOME/.local/scripts/fzf-git.bash"
  #fi

  ### git-fuzzy
  if [ -d "$HOME/.local/src/git-fuzzy/bin" ]; then
    export PATH="$HOME/.local/src/git-fuzzy/bin:$PATH"
  fi

  ### MOTD
  if [ -f "$XDG_CONFIG_HOME/shell/motd.sh" ]; then
    . "$XDG_CONFIG_HOME/shell/motd.sh"
  fi
fi

### Cargo
if [ -f "$CARGO_HOME/env" ]; then
  . "$CARGO_HOME/env"
fi

### Docker Functions / Aliases
# This file is used as an example file since it conflicts with the installation process of many libraries.
# Also, using Firejail is the preferred method of limiting the permissions of a process so there is no need
# to use Docker aliases since Firejail is superior (according to: https://news.ycombinator.com/item?id=21497677)
# if [ -f "$HOME/.local/scripts/docker-functions.bash" ]; then
#   . "$HOME/.local/scripts/docker-functions.bash"
# fi

### fzf-tmux
#if [ -f "$HOME/.local/scripts/fzf-tmux.bash" ]; then
#  . "$HOME/.local/scripts/fzf-tmux.bash"
#fi

### McFly
export MCFLY_PROMPT="❯"
if [ -d /Applications ] && [ -d /System ]; then
  if [[ "$(defaults read -g AppleInterfaceStyle 2&>/dev/null)" != "Dark" ]]; then
      export MCFLY_LIGHT=TRUE
  fi
fi

### SDKMan
if command -v brew > /dev/null && command -v sdkman-cli > /dev/null; then
  export SDKMAN_DIR="$(brew --prefix sdkman-cli)/libexec"
  . "$SDKMAN_DIR/bin/sdkman-init.sh"
elif [ -f "$SDKMAN_DIR/bin/sdkman-init.sh" ]; then
  export SDKMAN_DIR="$XDG_DATA_HOME/sdkman"
  . "$SDKMAN_DIR/bin/sdkman-init.sh"
fi

### VIM
export GVIMINIT='let $MYGVIMRC="$XDG_CONFIG_HOME/vim/gvimrc" | source $MYGVIMRC'
export VIMINIT='let $MYVIMRC="$XDG_CONFIG_HOME/vim/vimrc" | source $MYVIMRC'

### NVIM
VIM_NEOVIM_INTEGRATION='vim.cmd([[
set runtimepath^=~/.vim runtimepath+=~/.vim/after
let &packpath = &runtimepath
source ~/.vimrc
]])'
if command -v nvim > /dev/null && cat ~/.config/nvim/init.lua | grep 'set runtimepath^=~/.vim runtimepath+=~/.vim/after' > /dev/null; then
  ### Setup Neovim to work with Vim setup and plugins
  echo -e "$VIM_NEOVIM_INTEGRATION\n$(cat ~/.config/nvim/init.lua)" > ~/.config/nvim/init.lua || echo ''
  if -d ~/.var/app/io.neovim.nvim/config/nvim; then
    echo -e "$VIM_NEOVIM_INTEGRATION\n$(cat ~/.var/app/io.neovim.nvim/config/nvim/init.lua)" > ~/.var/app/io.neovim.nvim/config/nvim/init.lua || echo ''
  fi
fi
unset VIM_NEOVIM_INTEGRATION
```
