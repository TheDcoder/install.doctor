---
title: macOS Ensure User Group
description: Ensures that the provisioning user has a group on the system with the same name
sidebar_label: 20 macOS Ensure User Group
slug: /scripts/before/run_onchange_before_20-ensure-user-group.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_20-ensure-user-group.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_20-ensure-user-group.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_20-ensure-user-group.sh.tmpl
---
# macOS Ensure User Group

Ensures that the provisioning user has a group on the system with the same name

## Overview

This script ensures that there is a group with the same name of the provisioning user available on the system.



## Source Code

```
{{- if (eq .host.distro.family "darwin") -}}
#!/usr/bin/env bash
# @file macOS Ensure User Group
# @brief Ensures that the provisioning user has a group on the system with the same name
# @description
#     This script ensures that there is a group with the same name of the provisioning user available on the system.

{{ includeTemplate "universal/profile-before" }}
{{ includeTemplate "universal/logg-before" }}

if [ -n "$USER" ]; then
    logg info "Adding the $USER user to the $USER group"
    ### Ensure user has group of same name (required for Macports)
    logg info "Ensuring user ($USER) has a group with the same name ($USER) and that it is a member. Sudo privileges may be required"

    GROUP="$USER"
    USERNAME="$USER"

    ### Add group
    sudo dscl . create /Groups/$GROUP

    ### Add GroupID to group
    if [[ "$(sudo dscl . read /Groups/$GROUP gid 2>&1)" == *'No such key'* ]]; then
        MAX_ID_GROUP="$(dscl . -list /Groups gid | awk '{print $2}' | sort -ug | tail -1)"
        GROUP_ID="$((MAX_ID_GROUP+1))"
        sudo dscl . create /Groups/$GROUP gid "$GROUP_ID"
    fi

    ### Add user
    sudo dscl . create /Users/$USERNAME

    ### Add PrimaryGroupID to user
    if [[ "$(sudo dscl . read /Users/$USERNAME PrimaryGroupID 2>&1)" == *'No such key'* ]]; then
        sudo dscl . create /Users/$USERNAME PrimaryGroupID 20
    fi

    ### Add UniqueID to user
    if [[ "$(sudo dscl . read /Users/$USERNAME UniqueID 2>&1)" == *'No such key'* ]]; then
        MAX_ID_USER="$(dscl . -list /Users UniqueID | awk '{print $2}' | sort -ug | tail -1)"
        USER_ID="$((MAX_ID_USER+1))"
        sudo dscl . create /Users/$USERNAME UniqueID "$USERID"
    fi

    ### Add user to group
    sudo dseditgroup -o edit -t user -a $USERNAME $GROUP
else
    logg warn 'The USER environment variable is unavailable'
fi
{{ end -}}
```
