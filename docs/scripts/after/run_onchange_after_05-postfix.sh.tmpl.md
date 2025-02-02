---
title: SendGrid Postfix Configuration
description: Configures Postfix to use SendGrid as a relay host so you can use the `mail` program to send e-mail from the command-line
sidebar_label: 05 SendGrid Postfix Configuration
slug: /scripts/after/run_onchange_after_05-postfix.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_05-postfix.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_05-postfix.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_05-postfix.sh.tmpl
---
# SendGrid Postfix Configuration

Configures Postfix to use SendGrid as a relay host so you can use the `mail` program to send e-mail from the command-line

## Overview

This script follows the instructions from [SendGrid's documentation on integrating Postfix](https://docs.sendgrid.com/for-developers/sending-email/postfix).
After this script runs, you should be able to send outgoing e-mails using SendGrid as an SMTP handler. In other words, you will
be able to use the `mail` CLI command to send e-mails. The following is an example mailing the contents of `~/.bashrc` to `name@email.com`:

```shell
cat ~/.bashrc | mail -s "My subject" name@email.com
```



## Source Code

```
{{- if or (and (ne .host.distro.family "windows") (stat (joinPath .host.home ".config" "age" "chezmoi.txt")) (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "SENDGRID_API_KEY")) (env "SENDGRID_API_KEY")) -}}
#!/usr/bin/env bash
# @file SendGrid Postfix Configuration
# @brief Configures Postfix to use SendGrid as a relay host so you can use the `mail` program to send e-mail from the command-line
# @description
#     This script follows the instructions from [SendGrid's documentation on integrating Postfix](https://docs.sendgrid.com/for-developers/sending-email/postfix).
#     After this script runs, you should be able to send outgoing e-mails using SendGrid as an SMTP handler. In other words, you will
#     be able to use the `mail` CLI command to send e-mails. The following is an example mailing the contents of `~/.bashrc` to `name@email.com`:
#
#     ```shell
#     cat ~/.bashrc | mail -s "My subject" name@email.com
#     ```

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

if command -v postfix > /dev/null; then
    ### Ensure dependencies are installed
    if command -v apt > /dev/null; then
        logg info 'Installing libsasl2-modules'
        sudo apt-get update
        sudo apt-get install -y libsasl2-modules || EXIT_CODE=$?
    elif command -v dnf > /dev/null; then
        sudo dnf install -y cyrus-sasl-plain || EXIT_CODE=$?
    elif command -v yum > /dev/null; then
        sudo yum install -y cyrus-sasl-plain || EXIT_CODE=$?
    fi
    if [ -n "$EXIT_CODE" ]; then
        logg warn 'There was an error ensuring the Postfix-SendGrid dependencies were installed'
    fi

    if [ -d /etc/postfix ]; then
        ### Add the SendGrid Postfix settings to the Postfix configuration
        if [ -f "${XDG_CONFIG_HOME:-$HOME/.config}/postfix/main.cf" ]; then
            CONFIG_FILE=/etc/postfix/main.cf
            if cat "$CONFIG_FILE" | grep '### INSTALL DOCTOR MANAGED'; then
                logg info 'Removing Install Doctor-managed block of code in /etc/postfix/main.cf block'
                START_LINE="$(echo `grep -n -m 1 "### INSTALL DOCTOR MANAGED ### START" "$CONFIG_FILE" | cut -f1 -d ":"`)"
                END_LINE="$(echo `grep -n -m 1 "### INSTALL DOCTOR MANAGED ### END" "$CONFIG_FILE" | cut -f1 -d ":"`)"
                if command -v gsed > /dev/null; then
                    gsed -i "$START_LINE,$END_LINEd" "$CONFIG_FILE"
                else
                    sed -i "$START_LINE,$END_LINEd" "$CONFIG_FILE"
                fi
            fi

            ### Add Postfix main configuration
            logg "Adding configuration from ${XDG_CONFIG_HOME:-$HOME/.config}/postfix/main.cf to /etc/postfix/main.cf"
            echo "" | sudo tee -a "$CONFIG_FILE"
            cat "${XDG_CONFIG_HOME:-$HOME/.config}/postfix/main.cf" | sudo tee -a "$CONFIG_FILE"
        fi

        ### Ensure proper permissions on `sasl_passwd` and update Postfix hashmaps
        if [ -f "${XDG_CONFIG_HOME:-$HOME/.config}/postfix/sasl_passwd" ]; then
            logg info "Copying file from ${XDG_CONFIG_HOME:-$HOME/.config}/postfix/sasl_passwd to /etc/postfix/sasl_passwd"
            sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/postfix/sasl_passwd" /etc/postfix/sasl_passwd
            logg info 'Assigning proper permissions to /etc/postfix/sasl_passwd'
            sudo chmod 600 /etc/postfix/sasl_passwd
            logg info 'Updating Postfix hashmaps for /etc/postfix/sasl_passwd'
            sudo postmap /etc/postfix/sasl_passwd
        fi

        if [ -d /Applications ] && [ -d /System ]; then
            ### macOS
            # Source: https://budiirawan.com/install-mail-server-mac-osx/
            if [ -f "${XDG_CONFIG_HOME:-$HOME/.config}/postfix/com.apple.postfix.master.plist" ]; then
                logg info 'Copying com.apple.postfix.master.plist'
                sudo cp -f "${XDG_CONFIG_HOME:-$HOME/.config}/postfix/com.apple.postfix.master.plist" /System/Library/LaunchDaemons/com.apple.postfix.master.plist
            fi
            logg info 'Starting postfix'
            sudo postfix start
            logg info 'Reloading postfix'
            sudo postfix reload
        else
            ### Enable / restart postfix on Linux
            logg info 'Enabling / restarting postfix'
            sudo systemctl enable postfix
            sudo systemctl restart postfix
        fi
    else
        logg warn '/etc/postfix is not a directory! Skipping SendGrid Postfix setup.'
    fi
else
    logg info 'Skipping Postfix configuration because Postfix is not installed'
fi

{{ end -}}```
