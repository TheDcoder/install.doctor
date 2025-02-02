---
title: Aliases
description: Houses the aliases that are included by `~/.bashrc` and `~/.zshrc`
sidebar_label: Aliases
slug: /scripts/profile/aliases.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/shell/aliases.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/dot_config/shell/aliases.sh.tmpl
repoLocation: home/dot_config/shell/aliases.sh.tmpl
---
# Aliases

Houses the aliases that are included by `~/.bashrc` and `~/.zshrc`

## Overview

This script is included by `~/.bashrc` and `~/.zshrc` to provide command aliases.



## Source Code

```
#!/usr/bin/env sh
# @file Aliases
# @brief Houses the aliases that are included by `~/.bashrc` and `~/.zshrc`
# @description
#     This script is included by `~/.bashrc` and `~/.zshrc` to provide command aliases.

{{ if eq .chezmoi.os "darwin" }}
### macOS Polyfills
# Note: May cause conflicts
if command -v brew > /dev/null; then
  PATH="$(brew --prefix)/opt/coreutils/libexec/gnubin:$PATH"
  PATH="$(brew --prefix)/opt/gnu-indent/libexec/gnubin:$PATH"
  PATH="$(brew --prefix)/opt/gnu-sed/libexec/gnubin:$PATH"
fi

{{- end }}
# Basic command aliases for verbosity / simplicity
alias cp='cp -v'
alias ln='ln -sriv'
alias mv='mv -vi'
alias rm='rm -vi'

### Colorize
alias grep='grep --color=auto'
alias fgrep='fgrep --color=auto'
alias egrep='egrep --color=auto'
alias diff='diff --color=auto'
alias ip='ip --color=auto'
alias pacman='pacman --color=auto'

### TOP - order based on preference of "top" application (last item will always be chosen if installed, e.g. glances)
if command -v glances > /dev/null; then
  alias top='glances'
elif command -v htop > /dev/null; then
  alias top='bashtop'
fi

### bat
if command -v bat > /dev/null; then
  export MANPAGER="sh -c 'col -bx | bat -l man -p'"
  alias bathelp='bat --plain --language=help'
  alias cat='bat -pp'
  alias less='bat --paging=always'
  help() {
    "$@" --help 2>&1 | bathelp
  }
fi

### curlie
if command -v curlie > /dev/null; then
  alias curl='curlie'
fi

### exa
if command -v exa > /dev/null; then
  alias ls='exa --long --all --color auto --icons --sort=type'
  alias tree='exa --tree'
  alias la='ls -la'
  alias lt='ls --tree --level=2'
else
  # Show full output when using ls
  alias ls='ls -AlhF --color=auto'
fi

### gping
# Replacement for ping that includes graph
if command -v gping > /dev/null; then
  alias ping='gping'
fi

### VIM
if command -v vim > /dev/null; then
  alias vi='vim'
  alias v='vim'
fi
### NVIM
if command -v nvim > /dev/null; then
  alias nvim='env -u VIMINIT nvim'
fi

### mitmproxy / mitmweb
if command -v mitmproxy > /dev/null; then
  alias mitmproxy="mitmproxy --set confdir=$XDG_CONFIG_HOME/mitmproxy"
fi
if command -v mitmweb > /dev/null; then
  alias mitmweb="mitmweb --set confdir=$XDG_CONFIG_HOME/mitmproxy"
fi

### ripgrep
if command -v rg &> /dev/null; then
  alias rgrep='rg --color=auto'
fi

### xclip
alias xclip='xclip -selection c'

### Zola
if command -v org.getzola.zola > /dev/null; then
  alias zola="flatpak run org.getzola.zola"
fi

# Fix for auto expansion (source: https://wiki.archlinux.org/title/Sudo#Passing_aliases)
alias sudo='sudo '

# Reload current shell
alias reload="exec ${SHELL} -l"

# Create an Authelia password hash
alias authelia-password='docker run authelia/authelia:latest authelia hash-password'

# Shows IP addresses that are currently banned by fail2ban
alias banned='sudo zgrep "Ban" /var/log/fail2ban.log*'

alias connections='nm-connection-editor'

# Command-line DNS utility
if ! command -v dog > /dev/null; then
  alias dog="docker run -it --rm dog"
fi

# Download a file
alias download='curl --continue-at - --location --progress-bar --remote-name --remote-time'

# Download a website
alias download-site='wget --mirror -p --convert-links -P'

# Flush DNS
alias flush-dns='sudo systemd-resolve --flush-caches && sudo systemd-resolve --statistics'

# FontBook for macOS
alias fontbook="open -b com.apple.FontBook"

# Get the possible GRUB resolutions
alias grub-resolutions='sudo hwinfo --framebuffer'

# Execute git command with sudo priviledges while retaining .gitconfig
alias gsudo='sudo git -c "include.path="${XDG_CONFIG_DIR:-$HOME/.config}/git/config\" -c \"include.path=$HOME/.gitconfig\"'

# Create hashed password for Ansible user creation
alias hash-password='mkpasswd --method=sha-512'

# Show IP address
alias ip-address='curl http://ipecho.net/plain; echo'

# Shows local IP addresses
alias local-ip-address="ifconfig | grep -Eo 'inet (addr:|adr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1'"

# Create parent directories automatically
alias mkdir='mkdir -pv'

# Make mount command output readable
alias mount='mount | column -t'

# Link pip to pip3
if ! command -v pip > /dev/null; then
  alias pip='pip3'
fi

# Masked sudo password entry
if command -v gum > /dev/null; then
  alias please="gum input --password | sudo -nS"
fi

# Convert macOS plist to XML (for source control)
alias plist-xml='plutil -convert xml1'

# Recoverpy
alias recoverpy='python3 -m recoverpy'

# Show open ports
alias ports='sudo netstat -tulanp'

# Shuts down the computer, skipping the shutdown scripts
alias poweroff='sudo /sbin/poweroff'

# Open the Rclone web GUI
alias rclone-gui='rclone rcd --rc-web-gui --rc-user=admin --rc-pass=pass --rc-serve'

# Reboot the computer
alias reboot='sudo /sbin/reboot'

# Launch the Python Simple HTTP Server
alias serve='python -m SimpleHTTPServer'

# Generate a SHA1 digest
alias sha1='openssl sha1'

# Generate SHA256 digest
alias sha256='openssl sha256'

# Shutdown the computer
alias shutdown='sudo /sbin/shutdown'

# Speed test
alias speedtest='wget -O /dev/null http://speedtest.wdc01.softlayer.com/downloads/test10.zip'

# Shortcut for config file
alias ssh-config='${EDITOR:code} ~/.ssh/config'

# Pastebin
alias sprunge='curl -F "sprunge=<-" http://sprunge.us'

# Disable Tor for current shell
alias toroff='source torsocks off'

# Enable Tor for current shell
alias toron='source torsocks on'

# Test Tor connection
alias tortest='curl --socks5-hostname 127.0.0.1:9050 --silent https://check.torproject.org/  | head -25'

# Unban IP address (e.g. unban 10.14.24.14)
alias unban='sudo fail2ban-client set sshd unbanip'

# Recursively encrypts files using Ansible Vault
alias unvault-dir='find . -type f -printf "%h/\"%f\" " | xargs ansible-vault decrypt'

# Alias for updating software
if command -v sysget > /dev/null; then
  alias upgrade='sudo sysget update && sudo sysget upgrade'
else
  # TODO - Add other package managers
  if command -v apt-get > /dev/null; then
    alias upgrade='sudo apt-get update && sudo apt-get upgrade'
  fi
fi

# Recursively encrypts files using Ansible Vault
alias vault-dir='find . -type f -printf "%h/\"%f\" " | xargs ansible-vault encrypt'

# Shows nice looking report of weather
alias weather='curl -A curl wttr.in'

# Change .wget-hsts file location
alias wget="wget --hsts-file ~/.local/wget-hsts"

### Yarn
alias yarn='yarn --use-yarnrc "$XDG_CONFIG_HOME/yarn/config"'

# Running this will update GPG to point to the current YubiKey
alias yubi-stub='gpg-connect-agent "scd serialno" "learn --force" /bye'
```
