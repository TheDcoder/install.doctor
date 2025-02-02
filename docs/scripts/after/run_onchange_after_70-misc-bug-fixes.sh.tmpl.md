---
title: Miscellaneous Bug Fixes
description: This script applies miscellaneous bug fixes.
sidebar_label: 70 Miscellaneous Bug Fixes
slug: /scripts/after/run_onchange_after_70-misc-bug-fixes.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_70-misc-bug-fixes.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_70-misc-bug-fixes.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_70-misc-bug-fixes.sh.tmpl
---
# Miscellaneous Bug Fixes

This script applies miscellaneous bug fixes.

## Overview

This script houses bug fixes that do not yet have their own script file.



## Source Code

```
{{- if (eq .host.distro.family "linux") -}}
#!/usr/bin/env bash
# @file Miscellaneous Bug Fixes
# @brief This script applies miscellaneous bug fixes.
# @description
#     This script houses bug fixes that do not yet have their own script file.

# enabled extensions: {{ output "dconf" "read" "/org/gnome/shell/enabled-extensions" }}

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

### Remove Ubuntu logo branding from GDM lock screen
if [ '{{ .whiteLabel }}' == 'true' ]; then
    if [ -f /usr/share/plymouth/ubuntu-logo.png ]; then
        logg info 'Renaming `/usr/share/plymouth/ubuntu-logo.png` to `/usr/share/plymouth/ubuntu-logo.png.bak` since the whiteLabel setting is true'
        sudo mv /usr/share/plymouth/ubuntu-logo.png /usr/share/plymouth/ubuntu-logo.png.bak
    fi
fi

### Fix for Ubuntu default extension conflicting with dash-to-dock
if dconf read /org/gnome/shell/enabled-extensions | grep dash-to-dock > /dev/null; then
    if [ -d '/usr/share/gnome-shell/extensions/ubuntu-dock@ubuntu.com' ]; then
        if [ ! -d /usr/share/gnome-shell/extensions/disabled ]; then
            sudo mkdir /usr/share/gnome-shell/extensions/disabled
            logg info 'Created /usr/share/gnome-shell/extensions/disabled for GNOME extensions that have issues'
        fi
        sudo mv '/usr/share/gnome-shell/extensions/ubuntu-dock@ubuntu.com' '/usr/share/gnome-shell/extensions/disabled/ubuntu-dock@ubuntu.com'
        logg info 'Moved ubuntu-dock@ubuntu.com to the disabled extension folder'
    fi
fi

### Merge latest Candy icons into the Betelgeuse icon theme
if command -v rsync > /dev/null; then
    if [ -d "$HOME/.local/src/candy-icons" ] && [ -d /usr/local/share/icons/Candy ]; then
        sudo rsync --chown=root:root --chmod=Du=rwx,Dg=rx,Do=rx,Fu=rw,Fg=r,Fo=r -artu --inplace --exclude .git/ "$HOME/.local/src/candy-icons/" "/usr/local/share/icons/Candy/"
    else
        logg warn 'Skipping synchronization of Candy icons since either the target or destination folder is not present'
    fi
else
    logg warn '`rsync` is missing from the system!'
fi

### Move ~/.gnome/apps/* to ~/.local/share/applications
if [ -d "$HOME/.gnome/apps" ]; then
    if [ ! -d "${XDG_DATA_HOME:-$HOME/.local/share}/applications" ]; then
        mkdir -p "${XDG_DATA_HOME:-$HOME/.local/share}/applications"
    fi
    find "$HOME/.gnome/apps" -mindepth 1 -maxdepth 1 -type f | while read DESKTOP_FILE; do
        logg info "Moving $DESKTOP_FILE to ${XDG_DATA_HOME:-$HOME/.local/share}/applications/$(basename "$DESKTOP_FILE")"
        mv "$DESKTOP_FILE" "${XDG_DATA_HOME:-$HOME/.local/share}/applications/$(basename "$DESKTOP_FILE")"
        chmod 755 "${XDG_DATA_HOME:-$HOME/.local/share}/applications/$(basename "$DESKTOP_FILE")"
    done
    logg info 'Removing ~/.gnome/apps'
    rm -rf "$HOME/.gnome/apps"
fi

##### CANDY ICONS START ######

### Additional icons
SOURCE_DIR="/usr/local/share/icons/Candy/apps/scalable"
TARGET_DIR="/usr/local/share/icons/Candy/apps/scalable"
if [ -d "$SOURCE_DIR" ] && [ -d "$TARGET_DIR" ]; then
    logg info 'Adding similar substitutes for some apps in the Candy icons theme'
    if [ -f "$SOURCE_DIR/youtube-dl-gui.svg" ] && [ ! -f "$TARGET_DIR/com.github.Johnn3y.Forklift.svg" ]; then
        sudo cp -f "$SOURCE_DIR/youtube-dl-gui.svg" "$TARGET_DIR/com.github.Johnn3y.Forklift.svg"
    fi
    if [ -f "$SOURCE_DIR/rdm.svg" ] && [ ! -f "$TARGET_DIR/app.resp.RESP.svg" ]; then
        sudo cp -f "$SOURCE_DIR/rdm.svg" "$TARGET_DIR/app.resp.RESP.svg"
    fi
    if [ -f "$SOURCE_DIR/preferences-system-power.svg" ] && [ ! -f "$TARGET_DIR/org.gnome.PowerStats.svg" ]; then
        sudo cp -f "$SOURCE_DIR/preferences-system-power.svg" "$TARGET_DIR/org.gnome.PowerStats.svg"
    fi
    if [ -f "$SOURCE_DIR/software-store.svg" ] && [ ! -f "$TARGET_DIR/software-properties-gtk.svg" ]; then
        sudo cp -f "$SOURCE_DIR/software-store.svg" "$TARGET_DIR/software-properties-gtk.svg"
    fi
    if [ -f "$SOURCE_DIR/preferences-desktop-remote-desktop.svg" ] && [ ! -f "$TARGET_DIR/org.gnome.Connections.svg" ]; then
        sudo cp -f "$SOURCE_DIR/preferences-desktop-remote-desktop.svg" "$TARGET_DIR/org.gnome.Connections.svg"
    fi
    if [ -f "$SOURCE_DIR/com.github.wwmm.pulseeffects.svg" ] && [ ! -f "$TARGET_DIR/com.mattjakeman.ExtensionManager.svg" ]; then
        sudo cp -f "$SOURCE_DIR/com.github.wwmm.pulseeffects.svg" "$TARGET_DIR/com.mattjakeman.ExtensionManager.svg"
    fi
fi

### Icons added to fork (https://github.com/ProfessorManhattan/candy-icons)
# These commented out icons already had good matches in the Sweet theme so a fork was created
# and a pull request was open for them.
# sudo cp -f "$SOURCE_DIR/gitkraken.svg" "$TARGET_DIR/com.axosoft.GitKraken.svg"
# sudo cp -f "$SOURCE_DIR/github-desktop.svg" "$TARGET_DIR/io.github.shiftey.Desktop"
# sudo cp -f "$SOURCE_DIR/inkscape.svg" "$TARGET_DIR/inkscape_inkscape.desktop"
# sudo cp -f "$SOURCE_DIR/cutter.svg" "$TARGET_DIR/re.rizin.cutter.svg"
# sudo cp -f "$SOURCE_DIR/arduino.svg" "$TARGET_DIR/cc.arduino.IDE2.svg"    
# sudo cp -f "$SOURCE_DIR/intellij.svg" "$TARGET_DIR/intellij-idea-community_intellij-idea-community.svg"
# sudo cp -f "$SOURCE_DIR/google-chrome.svg" "$TARGET_DIR/com.google.Chrome.svg"
# sudo cp -f "$SOURCE_DIR/firefox.svg" "$TARGET_DIR/org.mozilla.firefox.svg"
# sudo cp -f "$SOURCE_DIR/microsoft-edge.svg" "$TARGET_DIR/com.microsoft.Edge.svg"
# sudo cp -f "$SOURCE_DIR/thunderbird.svg" "$TARGET_DIR/org.mozilla.Thunderbird.svg"
# sudo cp -f "$SOURCE_DIR/postman.svg" "$TARGET_DIR/com.getpostman.Postman.svg"
# sudo cp -f "$SOURCE_DIR/plexhometheater.svg" "$TARGET_DIR/tv.plex.PlexDesktop.svg"
# sudo cp -f "$SOURCE_DIR/seafile.svg" "$TARGET_DIR/com.client.Seafile.svg"
# sudo cp -f "$SOURCE_DIR/com.github.gi_lom.dialect.svg" "$TARGET_DIR/app.drey.Dialect.svg"
sudo cp -f "$SOURCE_DIR/teamviewer.svg" "$TARGET_DIR/com.teamviewer.TeamViewer.svg"
sudo cp -f "$SOURCE_DIR/terminator.svg" "$TARGET_DIR/tabby.svg"

### Missing icons
# The following applications are missing icons after using the "Full" installer. The application name
# is listed. To the right of each hyphen is the name of the `.desktop` file.
# Webkit Font Generator - 
# Lepton - lepton_lepton
# scrcpygui
# scrcpy - scrcpy_scrcpy
# Shotcut - org.shotcut.Shotcut
# Kooha - io.github.seadve.Kooha
# Lens - kontena-lens_kontena-lens
# Proton Mail Bridge - ch.protonmail.protonmail-bridge
# Proton Import-Export app - ch.protonmail.protonmail-import-export-app
# MQTTX - com.emqx.MQTTX
# Mockoon - mockoon_mockoon
# PowerShell - powershell_powershell
# GNOME Network Displays - org.gnome.NetworkDisplays
# Cockpit Client - org.cockpit_project.CockpitClient
# Yubico Authenticator - com.yubico.yubioath
# OnlyKey App - onlykey-app_onlykey-app
# Gitter - im.gitter.Gitter
# Jitsi Meet - org.jitsi.jitsi-meet
# Keybase
# Nuclear - org.js.nuclear.Nuclear
# Motrix - net.agalwood.Motrix
# Raspberry Pi Imager - org.raspberrypi.rpi-imager
# Junction - re.sonny.Junction
# GNOME Extension Manager - com.mattjakeman.ExtensionManager
# Startup Applications
# Multipass - multipass_gui
# Portmaster - portmaster, portmaster_notifier
# GNOME Connections - org.gnome.Connections

### Copy Snap desktop links to ~/.local/share/applications to apply custom icons
find /var/lib/snapd/desktop/applications -mindepth 1 -maxdepth 1 -name "*.desktop" | while read DESKTOP_FILE; do
    DESKTOP_FILE_BASE="$(basename "$DESKTOP_FILE" | sed 's/.desktop$//')"
    SNAP_ICON_BASE="$(echo "$DESKTOP_FILE_BASE" | sed 's/^[^_]*_//')"
    if [ -f "/usr/local/share/icons/Candy/apps/scalable/${DESKTOP_FILE_BASE}.svg" ] || [ -f "/usr/local/share/icons/Candy/apps/scalable/${SNAP_ICON_BASE}.svg" ]; then
        logg info "Found matching Candy icon theme icon for $DESKTOP_FILE"
        if [ ! -f "${XDG_DATA_HOME:-$HOME/.local/share}/applications/${DESKTOP_FILE_BASE}.desktop" ]; then
            cp "$DESKTOP_FILE" "${XDG_DATA_HOME:-$HOME/.local/share}/applications"
            logg info "Copied the .desktop shortcut to ${XDG_DATA_HOME:-$HOME/.local/share}/applications"
            if [ -f "/usr/local/share/icons/Candy/apps/scalable/${SNAP_ICON_BASE}.svg" ]; then
                SNAP_ICON="${SNAP_ICON_BASE}"
            else
                SNAP_ICON="${DESKTOP_FILE_BASE}"
            fi
            logg info 'Setting the .desktop shortcut Icon value equal to `'"$SNAP_ICON"'`'
            sed -i 's/^Icon=.*$/Icon='"$SNAP_ICON"'/' "${XDG_DATA_HOME:-$HOME/.local/share}/applications/${DESKTOP_FILE_BASE}.desktop"
        else
            logg info "${XDG_DATA_HOME:-$HOME/.local/share}/applications/${DESKTOP_FILE_BASE}.desktop already exists!"
        fi
    fi
done

##### CANDY ICONS END ######

{{ end -}}
```
