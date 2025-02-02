---
title: Firefox Settings / Add-Ons / Profiles
description: This script configures system-wide settings, sets up Firefox Profile Switcher, creates various profiles from different sources, and installs a configurable list of Firefox Add-Ons.
sidebar_label: 40 Firefox Settings / Add-Ons / Profiles
slug: /scripts/after/run_onchange_after_40-firefox.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_40-firefox.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_40-firefox.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_40-firefox.sh.tmpl
---
# Firefox Settings / Add-Ons / Profiles

This script configures system-wide settings, sets up Firefox Profile Switcher, creates various profiles from different sources, and installs a configurable list of Firefox Add-Ons.

## Overview

The Firefox setup script performs a handful of tasks that automate the setup of Firefox as well as
useful utilities that will benefit Firefox power-users. The script also performs the same logic on
[LibreWolf](https://librewolf.net/) installations.

## Features

* Installs and sets up [Firefox Profile Switcher](https://github.com/null-dev/firefox-profile-switcher)
* Sets up system-wide enterprise settings (with configurations found in `~/.local/share/firefox`)
* Sets up a handful of default profiles to use with the Firefox Profile Switcher
* Automatically installs the plugins defined in the firefoxAddOns key of [`home/.chezmoidata.yaml`](https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoidata.yaml) to the Standard and Private profiles
* Configures the default profile to clone its settings from the profile stored in firefoxPublicProfile of `home/.chezmoidata.yaml`
* Optionally, if the Chezmoi encryption key is present, then the default profile will be set to the contents of an encrypted `.tar.gz` that must be stored in the cloud somewhere (with the firefoxPrivateProfile key in `home/.chezmoidata.yaml` defining the URL of the encrypted `.tar.gz`)

## Profiles

The script sets up numerous profiles for user flexibility. They can be switched by using the Firefox Profile Switcher
that this script sets up. The map of the profiles is generated by using the template file stored in `~/.local/share/firefox/profiles.ini`.
The following details the features of each profile:

| Name             | Description                                                                                 |
|------------------|---------------------------------------------------------------------------------------------|
| Factory          | Default browser settings (system-wide configurations still apply)                           |
| default-release  | Same as Factory (unmodified and generated by headlessly opening Firefox / LibreWolf)        |
| Git (Public)     | Pre-configured profile with address stored in `firefoxPublicProfile`                        |
| Standard         | Cloned from the profile above with `firefoxAddOns` also installed                           |
| Miscellaneous    | Cloned from the Factory profile (with the user.js found in `~/.config/firefox` applied)     |
| Development      | Same as Miscellaneous                                                                       |
| Automation       | Same as Miscellaneous                                                                       |
| Private          | Populated from an encrypted profile stored in the cloud (also installs `firefoxAddOns`)     |

## Notes

* The Firefox Profile Switcher is only compatible with Firefox and not LibreWolf
* This script is only designed to properly provision profiles on a fresh installation (so it does not mess around with pre-existing / already configured profiles)
* Additional profiles for LibreWolf are not added because the Firefox Profile Switcher is not compatible with LibreWolf

## Links

* [System-wide configurations](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_local/share/firefox) as well as the location of the `profile.ini` and some other configurations
* [User-specific configurations](https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/firefox/user.js) added to all profiles except Factory



## Source Code

```
{{- if ne .host.distro.family "windows" -}}
#!/usr/bin/env bash
# @file Firefox Settings / Add-Ons / Profiles
# @brief This script configures system-wide settings, sets up Firefox Profile Switcher, creates various profiles from different sources, and installs a configurable list of Firefox Add-Ons.
# @description
#     The Firefox setup script performs a handful of tasks that automate the setup of Firefox as well as
#     useful utilities that will benefit Firefox power-users. The script also performs the same logic on
#     [LibreWolf](https://librewolf.net/) installations.
#
#     ## Features
#
#     * Installs and sets up [Firefox Profile Switcher](https://github.com/null-dev/firefox-profile-switcher)
#     * Sets up system-wide enterprise settings (with configurations found in `~/.local/share/firefox`)
#     * Sets up a handful of default profiles to use with the Firefox Profile Switcher
#     * Automatically installs the plugins defined in the firefoxAddOns key of [`home/.chezmoidata.yaml`](https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoidata.yaml) to the Standard and Private profiles
#     * Configures the default profile to clone its settings from the profile stored in firefoxPublicProfile of `home/.chezmoidata.yaml`
#     * Optionally, if the Chezmoi encryption key is present, then the default profile will be set to the contents of an encrypted `.tar.gz` that must be stored in the cloud somewhere (with the firefoxPrivateProfile key in `home/.chezmoidata.yaml` defining the URL of the encrypted `.tar.gz`)
#
#     ## Profiles
#
#     The script sets up numerous profiles for user flexibility. They can be switched by using the Firefox Profile Switcher
#     that this script sets up. The map of the profiles is generated by using the template file stored in `~/.local/share/firefox/profiles.ini`.
#     The following details the features of each profile:
#
#     | Name             | Description                                                                                 |
#     |------------------|---------------------------------------------------------------------------------------------|
#     | Factory          | Default browser settings (system-wide configurations still apply)                           |
#     | default-release  | Same as Factory (unmodified and generated by headlessly opening Firefox / LibreWolf)        |
#     | Git (Public)     | Pre-configured profile with address stored in `firefoxPublicProfile`                        |
#     | Standard         | Cloned from the profile above with `firefoxAddOns` also installed                           |
#     | Miscellaneous    | Cloned from the Factory profile (with the user.js found in `~/.config/firefox` applied)     |
#     | Development      | Same as Miscellaneous                                                                       |
#     | Automation       | Same as Miscellaneous                                                                       |
#     | Private          | Populated from an encrypted profile stored in the cloud (also installs `firefoxAddOns`)     |
#
#     ## Notes
#
#     * The Firefox Profile Switcher is only compatible with Firefox and not LibreWolf
#     * This script is only designed to properly provision profiles on a fresh installation (so it does not mess around with pre-existing / already configured profiles)
#     * Additional profiles for LibreWolf are not added because the Firefox Profile Switcher is not compatible with LibreWolf
#
#     ## Links
#
#     * [System-wide configurations](https://github.com/megabyte-labs/install.doctor/tree/master/home/dot_local/share/firefox) as well as the location of the `profile.ini` and some other configurations
#     * [User-specific configurations](https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/firefox/user.js) added to all profiles except Factory

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

# Firefox plugins: {{ list (.firefoxAddOns | toString | replace "[" "" | replace "]" "") | uniq | join " " }}

### Installs the Firefox Profile Connector on Linux systems (Snap / Flatpak installs are not included in this function, but instead inline below)
function installFirefoxProfileConnector() {
    logg info 'Installing the Firefox Profile Connector'
    if command -v apt-get > /dev/null; then
        sudo apt-get install -y https://github.com/null-dev/firefox-profile-switcher-connector/releases/latest/download/linux-x64.deb
    elif command -v dnf > /dev/null; then
        sudo dnf install -y https://github.com/null-dev/firefox-profile-switcher-connector/releases/latest/download/linux-x64.rpm
    elif command -v yay > /dev/null; then
        yay -Ss firefox-profile-switcher-connector
    else
        logg warn 'apt-get, dnf, and yay were all unavailable so the Firefox Profile Connector helper executable could not be installed'
    fi
    
}

### Add Firefox enterprise profile
# TODO - figure out how to do this for other installations like Flatpak and macOS and Librewolf
for FIREFOX_DIR in '/usr/lib/firefox' '/usr/lib/firefox-esr' '/etc/firefox' '/etc/firefox-esr' '/Applications/Firefox.app/Contents/Resources' '/Applications/LibreWolf.app/Contents/Resources/'; do
    if [ -d "$FIREFOX_DIR" ] && [ -d "${XDG_DATA_HOME:-$HOME/.local/share}/firefox" ] && command -v rsync > /dev/null; then
        sudo rsync -artvu "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/" "$FIREFOX_DIR"
    fi
done

### Loop through various Firefox profile locations
for SETTINGS_DIR in "$HOME/snap/firefox/common/.mozilla/firefox" "$HOME/.var/app/org.mozilla.firefox/.mozilla/firefox" "$HOME/.var/app/io.gitlab.librewolf-community/.librewolf" "$HOME/Library/Application Support/Firefox/Profiles" "$HOME/Library/Application Support/LibreWolf/Profiles" "$HOME/.mozilla/firefox"; do
    ### Determine executable to use
    logg info "Processing Firefox profile location $SETTINGS_DIR"
    unset FIREFOX_EXE
    if [ "$SETTINGS_DIR" == "$HOME/.var/app/org.mozilla.firefox" ]; then
        if ! command -v org.mozilla.firefox > /dev/null; then
            continue
        else
            FIREFOX_EXE="$(which org.mozilla.firefox)"

            ### Firefox Profile Switcher
            BASE_DIR="$HOME/.var/app/org.mozilla.firefox"
            BIN_INSTALL_DIR="$BASE_DIR/data/firefoxprofileswitcher-install"
            MANIFEST_INSTALL_DIR="$BASE_DIR/.mozilla/native-messaging-hosts"
            DOWNLOAD_URL="https://github.com/null-dev/firefox-profile-switcher-connector/releases/latest/download/linux-x64.deb"

            ### Ensure Firefox Profile Switcher is not already installed
            if [ ! -f "$BIN_INSTALL_DIR/usr/bin/ff-pswitch-connector" ] || [ ! -f "$MANIFEST_INSTALL_DIR/ax.nd.profile_switcher_ff.json" ]; then
                ### Download profile switcher
                mkdir -p "$BIN_INSTALL_DIR"
                TMP_FILE="$(mktemp)"
                logg info 'Downloading Firefox Profile Switch connector'
                curl -sSL "$DOWNLOAD_URL" -o "$TMP_FILE"
                ar p "$TMP_FILE" data.tar.xz | tar xfJ - --strip-components=2 -C "$BIN_INSTALL_DIR" usr/bin/ff-pswitch-connector
                rm -f "$TMP_FILE"

                ### Create manifest
                logg info 'Copying profile switcher configuration to manifest directory'
                mkdir -p "$MANIFEST_INSTALL_DIR"
                cat "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/profile-switcher.json" | sed 's/PATH_PLACEHOLDER/'"$BIN_INSTALL_DIR"'/' > "$MANIFEST_INSTALL_DIR/ax.nd.profile_switcher_ff.json"
            fi
        fi
    elif [ "$SETTINGS_DIR" == "$HOME/.var/app/io.gitlab.librewolf-community/.librewolf" ]; then
        if ! command -v io.gitlab.librewolf-community > /dev/null; then
            continue
        else
            FIREFOX_EXE="$(which io.gitlab.librewolf-community)"
        fi
    elif [ "$SETTINGS_DIR" == "$HOME/Library/Application Support/Firefox/Profiles" ]; then
        FIREFOX_EXE="/Applications/Firefox.app/Contents/MacOS/firefox"
        if [ ! -f "$FIREFOX_EXE" ]; then
            continue
        else
            ### Download Firefox Profile Switcher
            if [ ! -d /usr/local/Cellar/firefox-profile-switcher-connector ]; then
                logg info 'Ensuring Firefox Profile Switcher is installed'
                brew install null-dev/firefox-profile-switcher/firefox-profile-switcher-connector
            fi

            ### Ensure Firefox Profile Switcher configuration is symlinked
            if [ ! -d "/Library/Application Support/Mozilla/NativeMessagingHosts/ax.nd.profile_switcher_ff.json" ]; then
                logg info 'Ensuring Firefox Profile Switcher is configured'
                sudo mkdir -p "/Library/Applcation Support/Mozilla/NativeMessagingHosts"
                sudo ln -sf "$(brew ls -l firefox-profile-switcher-connector | grep -i ax.nd.profile_switcher_ff.json | head -n1)" "/Library/Application Support/Mozilla/NativeMessagingHosts/ax.nd.profile_switcher_ff.json"
            fi
        fi
    elif [ "$SETTINGS_DIR" == "$HOME/Library/Application Support/LibreWolf/Profiles" ]; then
        FIREFOX_EXE="/Applications/LibreWolf.app/Contents/MacOS/librewolf"
        if [ ! -f "$FIREFOX_EXE" ]; then
            continue
        fi
    elif [ "$SETTINGS_DIR" == "$HOME/snap/firefox/common/.mozilla/firefox" ]; then
        FIREFOX_EXE="/snap/bin/firefox"
        if [ ! -f "$FIREFOX_EXE" ]; then
            continue
        else
            ### Firefox Profile Switcher
            BASE_DIR="$HOME/snap/firefox/common"
            BIN_INSTALL_DIR="$BASE_DIR/firefoxprofileswitcher-install"
            MANIFEST_INSTALL_DIR="$BASE_DIR/.mozilla/native-messaging-hosts"
            DOWNLOAD_URL="https://github.com/null-dev/firefox-profile-switcher-connector/releases/latest/download/linux-x64.deb"

            ### Ensure Firefox Profile Switcher is not already installed
            if [ ! -f "$BIN_INSTALL_DIR/usr/bin/ff-pswitch-connector" ] || [ ! -f "$MANIFEST_INSTALL_DIR/ax.nd.profile_switcher_ff.json" ]; then
                ### Download profile switcher
                mkdir -p "$BIN_INSTALL_DIR"
                TMP_FILE="$(mktemp)"
                logg info 'Downloading Firefox Profile Switch connector'
                curl -sSL "$DOWNLOAD_URL" -o "$TMP_FILE"
                ar p "$TMP_FILE" data.tar.xz | tar xfJ - --strip-components=2 -C "$BIN_INSTALL_DIR" usr/bin/ff-pswitch-connector
                rm -f "$TMP_FILE"

                ### Create manifest
                logg info 'Copying profile switcher configuration to manifest directory'
                mkdir -p "$MANIFEST_INSTALL_DIR"
                cat "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/profile-switcher.json" | sed 's/PATH_PLACEHOLDER/'"$BIN_INSTALL_DIR"'/' > "$MANIFEST_INSTALL_DIR/ax.nd.profile_switcher_ff.json"
            fi
        fi
    elif [ "$SETTINGS_DIR" == "$HOME/.mozilla/firefox" ]; then
        if command -v firefox-esr > /dev/null; then
            FIREFOX_EXE="$(which firefox-esr)"
            installFirefoxProfileConnector
        elif command -v firefox > /dev/null && [ "$(which firefox)" != *'snap'* ] && [ "$(which firefox)" != *'flatpak'* ] && [ ! -d /Applications ] && [ ! -d /System ]; then
            # Conditional check ensures Snap / Flatpak / macOS Firefox versions do not try to install to the wrong folder
            FIREFOX_EXE="$(which firefox)"
            installFirefoxProfileConnector
        else
            if [ -d /Applications ] && [ -d /System ]; then
                # Continue on macOS without logging because profiles are not stored here on macOS
                continue
            else
                logg warn 'Unable to register Firefox executable'
                logg info "Settings directory: $SETTINGS_DIR"
                continue
            fi
        fi
    fi
    ### Initiatize Firefox default profiles
    if command -v "$FIREFOX_EXE" > /dev/null; then
        ### Create default profile by launching Firefox headlessly
        logg info "Firefox executable set to $FIREFOX_EXE"
        if [ ! -d "$SETTINGS_DIR" ]; then
            logg info 'Running Firefox headlessly to generate default profiles'
            timeout 8 "$FIREFOX_EXE" --headless
            logg info 'Finished running Firefox headlessly'
        fi

        ### Add the populated profiles.ini
        logg info "Copying "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/profiles.ini" to profile directory"
        if [ -d /Applications ] && [ -d /System ]; then
            # macOS
            logg info "Copying ~/.local/share/profiles.ini to $SETTINGS_DIR/../profiles.ini"
            cp -f "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/profiles.ini" "$SETTINGS_DIR/../profiles.ini"
            SETTINGS_INI="$SETTINGS_DIR/../installs.ini"
        else
            # Linux
            logg info "Copying ~/.local/share/profiles.ini to $SETTINGS_DIR/profiles.ini"
            cp -f "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/profiles.ini" "$SETTINGS_DIR/profiles.ini"
            SETTINGS_INI="$SETTINGS_DIR/installs.ini"
        fi

        ### Default profile (created by launching Firefox headlessly)
        # DEFAULT_RELEASE_PROFILE="$(find "$SETTINGS_DIR" -mindepth 1 -maxdepth 1 -name "*.default-*")"
        DEFAULT_PROFILE_PROFILE="$SETTINGS_DIR/$(cat "$SETTINGS_INI" | grep 'Default=' | sed 's/.*Profiles\///')"
        logg info 'Removing previous installs.ini file'
        rm -f "$SETTINGS_INI"
        # DEFAULT_PROFILE="$(find "$SETTINGS_DIR" -mindepth 1 -maxdepth 1 -name "*.default" -not -name "profile.default")"
        if [ -n "$DEFAULT_RELEASE_PROFILE" ]; then
            logg info "Syncing $DEFAULT_RELEASE_PROFILE to $SETTINGS_DIR/profile.default"
            rsync -a "$DEFAULT_RELEASE_PROFILE/" "$SETTINGS_DIR/profile.default"
        fi

        ### Miscellaneous default profiles
        for NEW_PROFILE in "automation" "development" "miscellaneous"; do
            if [ ! -d "$SETTINGS_DIR/profile.${NEW_PROFILE}" ] && [ -d "$SETTINGS_DIR/profile.default" ]; then
                logg info "Cloning $NEW_PROFILE from profile.default"
                rsync -a "$SETTINGS_DIR/profile.default/" "$SETTINGS_DIR/profile.${NEW_PROFILE}"
                rsync -a "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/" "$SETTINGS_DIR/profile.${NEW_PROFILE}"
                cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/firefox/user.js" "$SETTINGS_DIR/profile.${NEW_PROFILE}"
            fi
        done

        ### Public git profile
        if [ -d "$SETTINGS_DIR/profile.git" ]; then
            logg info 'Resetting the Firefox git profile'
            cd "$SETTINGS_DIR/profile.git"
            git reset --hard HEAD
            git clean -fxd
            logg info 'Pulling latest updates to the Firefox git profile'
            git pull origin master
        else
            logg info 'Cloning the public Firefox git profile'
            cd "$SETTINGS_DIR"
            git clone {{ .firefoxPublicProfile }} profile.git
        fi

        ### Copy user.js to profile.git profile
        cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/firefox/user.js" "$SETTINGS_DIR/profile.git"

        ### Git profile w/ plugins installed (installation happens below)
        if [ ! -d "$SETTINGS_DIR/profile.plugins" ]; then
            logg info "Syncing $SETTINGS_DIR/profile.git to $SETTINGS_DIR/profile.plugins"
            rsync -a "$SETTINGS_DIR/profile.git/" "$SETTINGS_DIR/profile.plugins"
            rsync -a "${XDG_DATA_HOME:-$HOME/.local/share}/firefox/" "$SETTINGS_DIR/profile.plugins"
            cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/firefox/user.js" "$SETTINGS_DIR/profile.plugins"
        fi

        {{- if stat (joinPath .host.home ".config" "age" "chezmoi.txt") }}
        ### Private hosted profile
        # Deprecated in favor of using the Restic profile tasks saved in `~/.config/task/Taskfile.yml`
        # if [ ! -d "$SETTINGS_DIR/profile.private" ]; then
        #     logg info 'Downloading the encrypted Firefox private profile'
        #     cd "$SETTINGS_DIR"
        #     curl -sSL '{ { .firefoxPrivateProfile } }' -o profile.private.tar.gz.age
        #     logg info 'Decrypting the Firefox private profile'
        #     chezmoi decrypt profile.private.tar.gz.age > profile.private.tar.gz || EXIT_DECRYPT_CODE=$?
        #     if [ -z "$EXIT_DECRYPT_CODE" ]; then
        #         rm -f profile.private.tar.gz.age
        #         logg info 'Decompressing the Firefox private profile'
        #         tar -xzf profile.private.tar.gz
        #         logg success 'The Firefox private profile was successfully installed'
        #         cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/firefox/user.js" "$SETTINGS_DIR/profile.private"
        #         logg info 'Copied ~/.config/firefox/user.js to profile.private profile'
        #     else
        #         logg error 'Failed to decrypt the private Firefox profile'
        #     fi
        # fi
        {{- end }}
        
        ### Install Firefox addons (using list declared in .chezmoidata.yaml)
        for SETTINGS_PROFILE in "profile.plugins" "profile.private"; do
            if [ -d "$SETTINGS_DIR/$SETTINGS_PROFILE" ]; then
                for FIREFOX_PLUGIN in {{ list (.firefoxAddOns | toString | replace "[" "" | replace "]" "") | uniq | join " " }}; do
                    logg info "Processing the $FIREFOX_PLUGIN Firefox add-on"
                    PLUGIN_HTML="$(mktemp)"
                    curl --silent "https://addons.mozilla.org/en-US/firefox/addon/$FIREFOX_PLUGIN/" > "$PLUGIN_HTML"
                    PLUGIN_TMP="$(mktemp)"
                    cat "$PLUGIN_HTML" | htmlq '#redux-store-state' | sed 's/^<scri.*application\/json">//' | sed 's/<\/script>$//' > "$PLUGIN_TMP"
                    PLUGIN_ID="$(jq '.addons.bySlug["'"$FIREFOX_PLUGIN"'"]' "$PLUGIN_TMP")"
                    if [ "$PLUGIN_ID" != 'null' ]; then
                        PLUGIN_FILE_ID="$(jq -r '.addons.byID["'"$PLUGIN_ID"'"].guid' "$PLUGIN_TMP")"
                        if [ "$PLUGIN_FILE_ID" != 'null' ]; then
                            PLUGIN_URL="$(cat "$PLUGIN_HTML" | htmlq '.InstallButtonWrapper-download-link' --attribute href)"
                            PLUGIN_FILENAME="${PLUGIN_FILE_ID}.xpi"
                            PLUGIN_FOLDER="$(echo "$PLUGIN_FILENAME" | sed 's/.xpi$//')"
                            if [ ! -d "$SETTINGS_DIR/$SETTINGS_PROFILE/extensions/$PLUGIN_FOLDER" ]; then
                                logg info 'Downloading add-on XPI file for '"$PLUGIN_FILENAME"' ('"$FIREFOX_PLUGIN"')'
                                if [ ! -d "$SETTINGS_DIR/$SETTINGS_PROFILE/extensions" ]; then
                                    mkdir -p "$SETTINGS_DIR/$SETTINGS_PROFILE/extensions"
                                fi
                                curl -sSL "$PLUGIN_URL" -o "$SETTINGS_DIR/$SETTINGS_PROFILE/extensions/$PLUGIN_FILENAME"
                                # Unzipping like this causes Firefox to complain about unsigned plugins
                                # TODO - figure out how to headlessly enable the extensions in such a way that is compatible with Flatpak / Snap
                                # using the /usr/lib/firefox/distribution/policies.json works but this is not compatible with Flatpak / Snap out of the box
                                # it seems since they do not have access to the file system by default. Also, using the policies.json approach forces
                                # all Firefox profiles to use the same extensions. Ideally, we should find a way to enable the extensions scoped
                                # to the user profile.
                                # logg info 'Unzipping '"$PLUGIN_FILENAME"' ('"$FIREFOX_PLUGIN"')'
                                # unzip "$SETTINGS_DIR/$SETTINGS_PROFILE/extensions/$PLUGIN_FILENAME" -d "$SETTINGS_DIR/$SETTINGS_PROFILE/extensions/$PLUGIN_FOLDER"
                                logg success 'Installed `'"$FIREFOX_PLUGIN"'`'
                            fi
                        else
                            logg warn 'A null Firefox add-on filename was detected for `'"$FIREFOX_PLUGIN"'`'
                        fi
                    else
                        logg warn 'A null Firefox add-on ID was detected for `'"$FIREFOX_PLUGIN"'`'
                    fi
                done
            fi
        done
    fi
done

{{ end -}}
```
