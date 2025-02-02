---
title: GPG Configuration
description: Imports the public GPG key defined by the variable `KEYID` and then assigns it ultimate trust
sidebar_label: 91 GPG Configuration
slug: /scripts/before/run_onchange_before_91-configure-gpg.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_91-configure-gpg.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_91-configure-gpg.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_91-configure-gpg.sh.tmpl
---
# GPG Configuration

Imports the public GPG key defined by the variable `KEYID` and then assigns it ultimate trust

## Overview

This script imports your publicly hosted GPG key using `pgp.mit.edu` as the key host. It then assigns it
the ultimate trust level. It also downloads and configures GPG to use the configuration defined in `.config.gpg`
in the `home/.chezmoidata.yaml` file.



## Source Code

```
#!/usr/bin/env bash
# @file GPG Configuration
# @brief Imports the public GPG key defined by the variable `KEYID` and then assigns it ultimate trust
# @description
#     This script imports your publicly hosted GPG key using `pgp.mit.edu` as the key host. It then assigns it
#     the ultimate trust level. It also downloads and configures GPG to use the configuration defined in `.config.gpg`
#     in the `home/.chezmoidata.yaml` file.

{{ includeTemplate "universal/profile-before" }}
{{ includeTemplate "universal/logg-before" }}

KEYID="{{ .user.gpg.id }}"

if [ -n "$KEYID" ] && command -v gpg > /dev/null; then
  if [ ! -d "$HOME/.gnupg" ]; then
    mkdir "$HOME/.gnupg"
  fi
  chown "$(whoami)" "$HOME/.gnupg"
  chmod 700 "$HOME/.gnupg"
  chown -Rf "$(whoami)" "$HOME/.gnupg/"
  find "$HOME/.gnupg" -type f -exec chmod 600 {} \;
  find "$HOME/.gnupg" -type d -exec chmod 700 {} \;
  if [ ! -f "$HOME/.gnupg/gpg.conf" ]; then
    logg 'Downloading hardened gpg.conf file to ~/.gpnupg/gpg.conf'
    curl -sSL "{{ .config.gpg }}" > "$HOME/.gnupg/gpg.conf"
    chmod 600 "$HOME/.gnupg/gpg.conf"
  fi
  KEYID_TRIMMED="$(echo "$KEYID" | sed 's/^0x//')"
  if ! gpg --list-secret-keys --keyid-format=long | grep "$KEYID_TRIMMED" > /dev/null; then
    logg info 'Attempting to download the specified public GPG key (`{{ .user.gpg.id }}`) from public keyservers'
    sudo pkill dirmngr
    dirmngr --daemon --standard-resolver
    gpg --keyserver https://pgp.mit.edu --recv "$KEYID" || EXIT_CODE=$?
    if [ -n "$EXIT_CODE" ]; then
      logg info 'Non-zero exit code received when downloading public GPG key'
      gpg --keyserver hkps://pgp.mit.edu --recv "$KEYID" || EXIT_CODE=$?
      if [ -n "$EXIT_CODE" ]; then
        logg info 'Non-zero exit code received when trying to retrieve public user GPG key on hkps://pgp.mit.edu'
        gpgconf --kill dirmngr
        KEYID="${KEYID^^}"
        KEYID="$(echo "$KEYID" | sed 's/^0X/0x/')"
        if [ -f "$HOME/.gnupg/public/$KEYID.sig" ]; then
          gpg --import "$HOME/.gnupg/public/$KEYID.sig"
        fi
      else
        logg success 'Successfully imported configured public user GPG key'
      fi
    fi
  else
    logg info 'Key is already in keyring'
  fi
  logg 'Ensuring the trust of the provided public GPG key is set to maximum'
  echo -e "trust\n5\ny" | gpg --command-fd 0 --edit-key "$KEYID"
else
  logg warn '`gpg` appears to be unavailable. Is it installed and on the PATH?'
fi
```
