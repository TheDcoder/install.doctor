---
title: Linux GNOME / KDE Theme
description: Installs the Sweet-based theme for GNOME / KDE.
sidebar_label: 19 Linux GNOME / KDE Theme
slug: /scripts/after/run_onchange_after_19-theme-files.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_19-theme-files.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_19-theme-files.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_19-theme-files.sh.tmpl
---
# Linux GNOME / KDE Theme

Installs the Sweet-based theme for GNOME / KDE.

## Overview

This script applies various themes to Linux systems that use GNOME or KDE.



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file Linux GNOME / KDE Theme
# @brief Installs the Sweet-based theme for GNOME / KDE.
# @description
#     This script applies various themes to Linux systems that use GNOME or KDE.

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Ensure /usr/local/bin/squash-symlink is present
if [ ! -f /usr/local/bin/squash-symlink ] && [ -f "$HOME/.local/bin/squash-symlink" ]; then
    logg info 'Copying ~/.local/bin/squash-symlink to /usr/local/bin/squash-symlink'
    sudo cp -f "$HOME/.local/bin/squash-symlink" /usr/local/bin/squash-symlink
fi

### Clean up system theme settings
for ITEM_TO_BE_REMOVED in "/usr/share/backgrounds/images" "/usr/share/backgrounds/f32" "/usr/share/backgrounds/qubes" "/usr/share/wallpapers"; do
  if [ -d "$ITEM_TO_BE_REMOVED" ] || [ -f "$ITEM_TO_BE_REMOVED" ]; then
    sudo rm -rf "$ITEM_TO_BE_REMOVED"
    logg success "Removed $ITEM_TO_BE_REMOVED"
  fi
done

### Ensure /usr/local/share exists
if [ ! -d /usr/local/share ]; then
    sudo mkdir -p /usr/local/share
    logg success 'Created /usr/local/share'
fi

### Copy theme files over to /usr/local/share
if [ -d "$HOME/.local/src/{{ .theme | lower }}/share" ]; then
    logg info 'Copying ~/.local/src/{{ .theme | lower }}/share to /usr/local/share'
    sudo rsync --chown=root:root --chmod=Du=rwx,Dg=rx,Do=rx,Fu=rw,Fg=r,Fo=r -artvu --inplace "$HOME/.local/src/betelgeuse/share/" "/usr/local/share/" > /dev/null
else
    logg warn '~/.local/src/betelgeuse/share is missing'
fi

### Flatten GRUB theme files (i.e. convert symlinks to regular files)
if command -v squash-symlink > /dev/null; then
    logg info 'Converting /usr/local/share/grub symlinks to equivalent regular files'
    sudo find /usr/local/share/grub -type l -exec squash-symlink {} +
else
    logg warn '`squash-symlink` is not a script in the PATH'
fi

### Ensure /usr/share/backgrounds/default.png is deleted
if [ -f /usr/share/backgrounds/default.png ]; then
    sudo rm -f /usr/share/backgrounds/default.png
fi

### Add the default image symlink based on the OS
if [ '{{ .host.distro.id }}' == 'archlinux' ]; then
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse-Archlinux/contents/source.png /usr/share/backgrounds/default.png
elif [ '{{ .host.distro.id }}' == 'centos' ]; then
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse-CentOS/contents/source.png /usr/share/backgrounds/default.png
elif [ '{{ .host.distro.id }}' == 'darwin' ]; then
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse-macOS/contents/source.png /usr/share/backgrounds/default.png
elif [ '{{ .host.distro.id }}' == 'debian' ]; then
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse-Debian/contents/source.png /usr/share/backgrounds/default.png
elif [ '{{ .host.distro.id }}' == 'fedora' ]; then
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse-Fedora/contents/source.png /usr/share/backgrounds/default.png
elif [ '{{ .host.distro.id }}' == 'ubuntu' ]; then
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse-Ubuntu/contents/source.png /usr/share/backgrounds/default.png
elif [ '{{ .host.distro.id }}' == 'windows' ]; then
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse-Windows/contents/source.png /usr/share/backgrounds/default.png
else
    sudo ln -s /usr/local/share/wallpapers/Betelgeuse/contents/source.png /usr/share/backgrounds/default.png
fi

### Note: ubuntu-default-greyscale-wallpaper.png symlink to whiteish gray background

### Set appropriate platform-specific icon in plymouth theme
if [ -f '/usr/local/share/plymouth/themes/{{ .theme }}/icons/{{ .host.distro.id }}.png' ]; then
    sudo cp -f '/usr/local/share/plymouth/themes/{{ .theme }}/icons/{{ .host.distro.id }}.png' '/usr/local/share/plymouth/themes/{{ .theme }}/icon.png'
    logg success 'Added platform-specific icon to {{ .theme }} Plymouth theme'
else
    logg warn 'The `{{ .host.distro.id }}.png` icon is not available in the icons folder insider the {{ .theme }} Plymouth theme'
fi

{{ end -}}
```
