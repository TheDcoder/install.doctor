---
title: Environment Variables
description: Houses the environment variables that are included by `~/.bashrc` and `~/.zshrc`
sidebar_label: Environment Variables
slug: /scripts/profile/exports.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/shell/exports.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/dot_config/shell/exports.sh.tmpl
repoLocation: home/dot_config/shell/exports.sh.tmpl
---

# Environment Variables

Houses the environment variables that are included by `~/.bashrc` and `~/.zshrc`

## Overview

This script is included by `~/.bashrc` and `~/.zshrc` to provide environment variables that play harmoniously with
the default Install Doctor configurations.

## Source Code

```
#!/usr/bin/env sh
# @file Environment Variables
# @brief Houses the environment variables that are included by `~/.bashrc` and `~/.zshrc`
# @description
#     This script is included by `~/.bashrc` and `~/.zshrc` to provide environment variables that play harmoniously with
#     the default Install Doctor configurations.

### Disable Telemetry
export DO_NOT_TRACK=1

### XDG
# Source: # https://wiki.archlinux.org/index.php/XDG_Base_Directory
export XDG_CONFIG_HOME="$HOME/.config"
export XDG_CACHE_HOME="$HOME/.cache"
export XDG_DATA_HOME="$HOME/.local/share"
export XDG_STATE_HOME="$HOME/.local/state"
export XDG_RUNTIME_DIR=
{{- if eq .host.distro.family "darwin" -}}
  "$TMPDIR"
{{- else if not .host.headless -}}
  "/run/user/$(id -u)"
{{- else -}}
  "/tmp"
{{- end }}
{{ if not .host.headless }}
export XDG_MUSIC_DIR="$HOME/Music"
export XDG_VIDEOS_DIR="$HOME/Videos"
export XDG_DESKTOP_DIR="$HOME/Desktop"
export XDG_PICTURES_DIR="$HOME/Pictures"
export XDG_DOWNLOAD_DIR="$HOME/Downloads"
export XDG_DOCUMENTS_DIR="$HOME/Documents"
export XDG_TEMPLATES_DIR="$HOME/Templates"
export XDG_PUBLICSHARE_DIR="$HOME/Public"
{{ end }}

### Theme
export COLOR_SCHEME=dark
export GTK_RC_FILES="$XDG_CONFIG_HOME/gtk-1.0/gtkrc"
export GTK2_RC_FILES="$XDG_CONFIG_HOME/gtk-2.0/gtkrc"

### PATH
export PATH="/opt/local/bin:$PATH"
export PATH="$HOME/.local/bin:$PATH"
export PATH="$HOME/.local/bin/cask:$PATH"
export PATH="$HOME/.local/bin/docker:$PATH"
export PATH="$HOME/.local/bin/firejail:$PATH"
export PATH="$HOME/.local/bin/flatpak:$PATH"
export SSH_KEY_PATH="~/.ssh/id_rsa"

### Homebrew
export HOMEBREW_NO_ENV_HINTS=true
if [ -d /home/linuxbrew/.linuxbrew/bin ]; then
  export HOMEBREW_PREFIX="/home/linuxbrew/.linuxbrew"
  export HOMEBREW_CELLAR="/home/linuxbrew/.linuxbrew/Cellar"
  export HOMEBREW_REPOSITORY="/home/linuxbrew/.linuxbrew/Homebrew"
  export PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin${PATH+:$PATH}"
  export MANPATH="/home/linuxbrew/.linuxbrew/share/man${MANPATH+:$MANPATH}:"
  export INFOPATH="/home/linuxbrew/.linuxbrew/share/info:${INFOPATH:-}"
  export WHALEBREW_INSTALL_PATH="/home/linuxbrew/.linuxbrew/whalebrew"
elif [ -f /opt/homebrew/bin/brew ]; then
  eval $(/opt/homebrew/bin/brew shellenv)
elif [ -f /usr/local/bin/brew ]; then
  eval $(/usr/local/bin/brew shellenv)
fi

### PATH References
export PATH_TASK="$(which task)"

{{ if eq .host.distro.id "darwin" }}
# Adjustment for macOS
export WHALEBREW_INSTALL_PATH="/usr/local/bin"

### Android Studio
if [ -d ~/Library/Android ]; then
  export PATH="$PATH:~/Library/Android/sdk/cmdline-tools/latest/bin"
  export PATH="$PATH:~/Library/Android/sdk/platform-tools"
  export PATH="$PATH:~/Library/Android/sdk/tools/bin"
  export PATH="$PATH:~/Library/Android/sdk/tools"
fi
{{ end }}
export ANDROID_SDK_HOME="$XDG_DATA_HOME/android-sdk"

### Ansible
export ANSIBLE_CONFIG="$XDG_DATA_HOME/ansible/ansible.cfg"
export ANSIBLE_HOME="$XDG_DATA_HOME/ansible"

### Aqua
export AQUA_ROOT_DIR="$XDG_DATA_HOME/aqua"
export AQUA_GLOBAL_CONFIG="$XDG_CONFIG_HOME/aqua/aqua.yaml"
export PATH="${AQUA_ROOT_DIR:-${XDG_DATA_HOME:-$HOME/.local/share}/aquaproj-aqua}/bin:$PATH"

### ASDF
export ASDF_CONFIG_FILE="$XDG_CONFIG_HOME/asdf/asdfrc"
export ASDF_DIR="$XDG_DATA_HOME/asdf"
export ASDF_DATA_DIR="$ASDF_DIR"
export ASDF_CRATE_DEFAULT_PACKAGES_FILE="$XDG_CONFIG_HOME/asdf/default-cargo-pkgs"
export ASDF_GEM_DEFAULT_PACKAGES_FILE="$XDG_CONFIG_HOME/asdf/default-ruby-pkgs"
export ASDF_GOLANG_DEFAULT_PACKAGES_FILE="$XDG_CONFIG_HOME/asdf/default-golang-pkgs"
export ASDF_NPM_DEFAULT_PACKAGES_FILE="$XDG_CONFIG_HOME/asdf/default-npm-packages"
export ASDF_PYTHON_DEFAULT_PACKAGES_FILE="$XDG_CONFIG_HOME/asdf/default-python-pkgs"

### AWS CLI
export AWS_SHARED_CREDENTIALS_FILE="$XDG_CONFIG_HOME/aws/credentials"
export AWS_CONFIG_FILE="$XDG_CONFIG_HOME/aws/config"

### Azure CLI
export AZURE_CONFIG_DIR="$XDG_CONFIG_HOME/azure"

### bat
export BAT_CONFIG_PATH="$XDG_CONFIG_HOME/bat/config"

### Cargo
export CARGO_HOME="$XDG_DATA_HOME/cargo"
export PATH="$PATH:$CARGO_HOME/bin"

### Bash
export BASH_COMPLETION_USER_FILE="$XDG_CONFIG_HOME/bash-completion/bash_completion"

### Bash It
# Don't check mail when opening terminal.
unset MAILCHECK
export GIT_HOSTING='git@gitlab.com'
if command -v irssi > /dev/null; then
  export IRC_CLIENT='irssi'
fi
# Set this to the command you use for todo.txt-cli TODO -- figure this out with Standard Notes / Lepton / or nb
export TODO="t"
export SCM_CHECK=true

### Boringtun
if command -v boringtun-cli > /dev/null; then
  export WG_QUICK_USERSPACE_IMPLEMENTATION=boringtun-cli
fi

### BitWarden
# https://bitwarden.com/help/cli/#using-an-api-key
# BW_CLIENTID client_id
# BW_CLIENTSECRET

### Deta
export DETA_INSTALL="$XDG_DATA_HOME/deta"
export PATH="$PATH:$DETA_INSTALL/bin"

### Docker
export DOCKER_CONFIG="$XDG_CONFIG_HOME/docker"
export MACHINE_STORAGE_PATH="$XDG_DATA_HOME/docker-machine"

### Dotnet
export DOTNET_CLI_HOME="$XDG_CONFIG_HOME/dotnet"
if [ -d /Applications ] && [ -d /Library ]; then
  export DOTNET_ROOT="/usr/local/opt/dotnet/libexec"
elif [ -d /home/linuxbrew/.linuxbrew/opt/dotnet ]; then
  export DOTNET_ROOT="/home/linuxbrew/.linuxbrew/opt/dotnet/libexec"
fi

### Elastic Agent
# https://www.elastic.co/guide/en/fleet/current/agent-environment-variables.html#env-common-vars

### ffmpeg
export FFMPEG_DATADIR="$XDG_CONFIG_HOME/ffmpeg"

### Flatpak
if command -v flatpak > /dev/null; then
  FLATPAK_INSTALLATIONS="$(flatpak --installations)"
  export PATH="$FLATPAK_INSTALLATIONS/exports/bin:$PATH"
  export XDG_DATA_DIRS="$HOME/.local/share/flatpak/exports/share:$FLATPAK_INSTALLATIONS/exports/share:$XDG_DATA_DIRS"
fi

### fzf
if command -v fd > /dev/null; then
  export FZF_DEFAULT_COMMAND='fd --type f --strip-cwd-prefix --hidden --follow --exclude .git'
  export FZF_DEFAULT_OPTS='--layout=reverse --inline-info'
  export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
fi

### Git
export GIT_MERGE_AUTOEDIT=no

### gitfuzzy
if command -v delta > /dev/null; then
  export GF_BAT_STYLE=changes
  export GF_BAT_THEME=zenbur
  export GF_SNAPSHOT_DIRECTORY="$XDG_DATA_HOME/git-fuzzy-snapshots"
  export GF_PREFERRED_PAGER="delta --theme=gruvbox --highlight-removed -w __WIDTH__"
fi

### Go
export GOPATH="$XDG_DATA_HOME/go"
export GO111MODULE=on
export PATH="$PATH:${GOPATH}/bin"
if command -v go > /dev/null && which go | grep -q 'asdf' > /dev/null && command -v asdf > /dev/null; then
  GOROOT="$(asdf where golang)/go"
  export GOROOT
  export PATH="$PATH:${GOROOT}/bin"
elif command -v go > /dev/null && command -v brew > /dev/null; then
  GOROOT="$(brew --prefix go)/libexec"
  export GOROOT
  export PATH="$PATH:${GOROOT}/bin"
fi

### Gradle
export GRADLE_USER_HOME="$XDG_DATA_HOME/gradle"

### Homebrew
export HOMEBREW_BUNDLE_FILE="$XDG_CONFIG_HOME/Brewfile"
{{ if (and (eq .host.distro.family "darwin") (.host.restricted)) }}
export HOMEBREW_CASK_OPTS="--appdir=~/Applications"
{{ end }}

### HTTPie
export HTTPIE_CONFIG_DIR="$XDG_CONFIG_HOME/httpie"

### IPFS
export IPFS_PATH="$XDG_DATA_HOME/ipfs"

### Java
export _JAVA_OPTIONS=-Djava.util.prefs.userRoot="$XDG_CONFIG_HOME/java"

### k9s
export K9SCONFIG="$XDG_CONFIG_HOME/k9s"

### KDE
export KDEHOME="$XDG_CONFIG_HOME/kde"

### Kodi
export KODI_DATA="$XDG_DATA_HOME/kodi"

### Krew
export KREW_ROOT="$XDG_DATA_HOME/krew"

### Kube
export KUBECONFIG="$XDG_CONFIG_HOME/kube/config"

### Maven
export MAVEN_CONFIG="$XDG_CONFIG_HOME/maven/settings.xml"
alias mvn="mvn -s $MAVEN_CONFIG"

### McFly
export MCFLY_FUZZY=2
export MCFLY_RESULTS=14
export MCFLY_KEY_SCHEME=vim

### minikube
export MINIKUBE_HOME="$XDG_DATA_HOME/minikube"

### MySQL
export MYSQL_HISTFILE="$XDG_DATA_HOME/mysql_history"

### .netrc
export NETRC="$XDG_CONFIG_HOME/netrc"

### nnn
if command -v nnn > /dev/null; then
  alias n='nnn -de'
  alias N='sudo -E nnn -dH'
  alias nnn-install-plugins='curl -Ls https://raw.githubusercontent.com/jarun/nnn/master/plugins/getplugs | sh'
  export NNN_RCLONE='rclone mount --read-only --no-checksum'
  export NNN_SSHFS='sshfs -o reconnect,idmap=user,cache_timeout=3600'
fi

### Node.js
export NODE_REPL_HISTORY="$XDG_DATA_HOME/node_repl_history"

### NPM
export NPM_CONFIG_USERCONFIG="$XDG_CONFIG_HOME/npm/npmrc"

### NTL
export NTL_RUNNER="pnpm"
export NTL_RERUN_CACHE_DIR="$XDG_DATA_HOME/ntl"
export NTL_RERUN_CACHE_NAME="cache"
export NTL_RERUN_CACHE_MAX="100"

### NuGet
export NUGET_PACKAGES="$XDG_DATA_HOME/nuget"

### Parallels
export PARALLEL_HOME="$XDG_CONFIG_HOME/parallel"

### Pass
export PASSWORD_STORE_DIR="$XDG_DATA_HOME/pass"

### Poetry
export POETRY_HOME="$XDG_DATA_HOME/poetry"
export PATH="$POETRY_HOME/bin:$PATH"

### Postgres
export PSQLRC="$XDG_CONFIG_HOME/pg/psqlrc"
export PSQL_HISTORY="$XDG_STATE_HOME/psql_history"
export PGPASSFILE="$XDG_CONFIG_HOME/pg/pgpass"
export PGSERVICEFILE="$XDG_CONFIG_HOME/pg/pg_service.conf"

### PNPM
export PNPM_HOME="$XDG_DATA_HOME/pnpm"
export PATH="$PATH:$PNPM_HOME"

### Prettierd
# Specify location of the default Prettierd configuration
# export PRETTIERD_DEFAULT_CONFIG=""

### Readline
export INPUTRC="$XDG_CONFIG_HOME/readline/inputrc"

### Redis
export REDISCLI_HISTFILE="$XDG_DATA_HOME/redis/rediscli_history"
export REDISCLI_RCFILE="$XDG_CONFIG_HOME/redis/redisclirc"

### ripgrep
export RIPGREP_CONFIG_PATH="$XDG_CONFIG_HOME/ripgrep/config"

### Ruby
export GEM_HOME="$XDG_DATA_HOME/gems"
export PATH="$PATH:$GEM_HOME/bin"

### Rustup
export RUSTUP_HOME="$XDG_DATA_HOME/rustup"

### SDKMan
export SDKMAN_DIR="$XDG_DATA_HOME/sdkman"

### Vagrant
export VAGRANT_ALIAS_FILE="$XDG_CONFIG_HOME/vagrant/aliases"
export VAGRANT_DEFAULT_PROVIDER=virtualbox
export VAGRANT_HOME="$XDG_DATA_HOME/vagrant.d"

### Volta
export VOLTA_HOME="$XDG_DATA_HOME/volta"
export PATH="$VOLTA_HOME/bin:$PATH"

### Wakatime
export WAKATIME_HOME="$XDG_CONFIG_HOME/wakatime"

### wget
export WGETRC="$XDG_CONFIG_HOME/wget/wgetrc"

### Whalebrew
export WHALEBREW_CONFIG_DIR="$XDG_CONFIG_HOME/whalebrew"

### CloudFlare Wrangler
export WRANGLER_INSTALL_PATH="$XDG_DATA_HOME/wrangler"
export WRANGLER_HOME="$XDG_DATA_HOME/wrangler"

### Man pages
export LESS_TERMCAP_mb=$'\e[1;32m'
export LESS_TERMCAP_md=$'\e[1;32m'
export LESS_TERMCAP_me=$'\e[0m'
export LESS_TERMCAP_se=$'\e[0m'
export LESS_TERMCAP_so=$'\e[01;33m'
export LESS_TERMCAP_ue=$'\e[0m'
export LESS_TERMCAP_us=$'\e[1;4;31m'
export LESSHISTFILE=-
export MANPAGER="less -X"

### Magic Enter (ZSH)
export MAGIC_ENTER_GIT_COMMAND='git status -u .'
export MAGIC_ENTER_OTHER_COMMAND='ls -lh .'

### Line Wrap
setterm -linewrap on 2>/dev/null

### History
export HISTCONTROL=ignoreboth
export HISTSIZE=1000000000
export HISTFILESIZE=$HISTSIZE
export HIST_STAMPS=mm/dd/yyyy
export SAVEHIST=50000

### Editor
{{ if not .host.headless }}
if command -v codium > /dev/null; then
  export EDITOR='codium --wait'
  export VISUAL="$EDITOR"
elif command -v code > /dev/null; then
  export EDITOR='code --wait'
  export VISUAL="$EDITOR"
else
  # Source: https://unix.stackexchange.com/questions/4859/visual-vs-editor-what-s-the-difference
  export EDITOR='vi -e'
  if command -v nvim > /dev/null; then
    export VISUAL='nvim -e'
  else
    export VISUAL="$EDITOR"
  fi
fi
{{ else }}
export EDITOR='vi -e'
if command -v nvim > /dev/null; then
  export VISUAL='nvim -e'
  export PATH="$PATH:$HOME/.local/share/nvim/bin"
else
  export VISUAL="$EDITOR"
fi
{{ end }}

### Browser
export BROWSER=brave

### WSL
if [ -d /proc ] && [[ "$(grep Microsoft /proc/version > /dev/null)" ]]; then
  # Source: https://stackoverflow.com/questions/61110603/how-to-set-up-working-x11-forwarding-on-wsl2
  export LIBGL_ALWAYS_INDIRECT="1"
  export DISPLAY=$(ip route list default | awk '{print $3}'):0
  export BROWSER='/mnt/c/Program\ Files/BraveSoftware/Brave-Browser/Application/brave.exe'
fi
```
