---
title: Chezmoi-Age Secret Decryption
description: Ensures `age` is installed and then decrypts the `home/key.txt.age` file so that Chezmoi can utilize encrypted files
sidebar_label: 01 Chezmoi-Age Secret Decryption
slug: /scripts/before/run_before_01-decrypt-age-key.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_before_01-decrypt-age-key.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_before_01-decrypt-age-key.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_before_01-decrypt-age-key.sh.tmpl
---
# Chezmoi-Age Secret Decryption

Ensures `age` is installed and then decrypts the `home/key.txt.age` file so that Chezmoi can utilize encrypted files

## Overview

This script begins by ensuring `age` is installed, the defualt program Chezmoi utilizes for handling encryption.
The script then allows you to generate the decrypted `~/.config/age/chezmoi.txt` file by prompting you for the password
to `home/key.txt.age` which is the encrypted encryption key file for using Chezmoi to add encrypted files to your Install
Doctor fork. If no password is passed to the decryption password prompt, then all of the `encrypted_` files in the fork
are deleted so that Chezmoi does not try to decrypt files without a decryption key file.

## Headless Installs

If you do not want the script to prompt you for a password, then you can pass in an environment variable with
`HEADLESS_INSTALL=true`. This variable ensures that nothing requiring input from the user blocks the provisioning process.
If you want to automate a headless install that requires access to `encrypted_` files and encrypted variables, then
you can save the decrypted Age key to `~/.config/age/chezmoi.txt` prior to running `bash <(curl -sSL https://install.doctor/start)`.

## GPG

It is also possible to configure Chezmoi to utilize GPG instead of Age. This might be beneficial if you want to
use a smart card / YubiKey for hardware-backed encryption. Otherwise, Age is a great encryption tool.

## Notes

_It is possible that hardware-based smart-card-like GPG encryption might not work properly with Chezmoi yet.
Learned this by attempting to use a YubiKey GPG setup using [this guide](https://github.com/drduh/YubiKey-Guide) and trying to get it to work with Chezmoi._



## Source Code

```
#!/usr/bin/env bash
# @file Chezmoi-Age Secret Decryption
# @brief Ensures `age` is installed and then decrypts the `home/key.txt.age` file so that Chezmoi can utilize encrypted files
# @description
#     This script begins by ensuring `age` is installed, the defualt program Chezmoi utilizes for handling encryption.
#     The script then allows you to generate the decrypted `~/.config/age/chezmoi.txt` file by prompting you for the password
#     to `home/key.txt.age` which is the encrypted encryption key file for using Chezmoi to add encrypted files to your Install
#     Doctor fork. If no password is passed to the decryption password prompt, then all of the `encrypted_` files in the fork
#     are deleted so that Chezmoi does not try to decrypt files without a decryption key file.
#
#     ## Headless Installs
#
#     If you do not want the script to prompt you for a password, then you can pass in an environment variable with
#     `HEADLESS_INSTALL=true`. This variable ensures that nothing requiring input from the user blocks the provisioning process.
#     If you want to automate a headless install that requires access to `encrypted_` files and encrypted variables, then
#     you can save the decrypted Age key to `~/.config/age/chezmoi.txt` prior to running `bash <(curl -sSL https://install.doctor/start)`.
#
#     ## GPG
#
#     It is also possible to configure Chezmoi to utilize GPG instead of Age. This might be beneficial if you want to
#     use a smart card / YubiKey for hardware-backed encryption. Otherwise, Age is a great encryption tool.
#
#     ## Notes
#
#     _It is possible that hardware-based smart-card-like GPG encryption might not work properly with Chezmoi yet.
#     Learned this by attempting to use a YubiKey GPG setup using [this guide](https://github.com/drduh/YubiKey-Guide) and trying to get it to work with Chezmoi._

{{ includeTemplate "universal/logg-before" }}
{{ includeTemplate "universal/profile-before" }}

### Only run decryption process if HEADLESS_INSTALL variable is not set
if [ -z "$HEADLESS_INSTALL" ]; then
  ### Install Age via Homebrew if not present
  if ! command -v age > /dev/null; then
    if command -v brew > /dev/null; then
      logg info 'Running `brew install age`'
      brew install age
    else
      logg warn '`age` is not installed which is utilized in the decryption process'
    fi
  fi

  ### Decrypt private key if it is not already present
  if command -v age > /dev/null; then
    if [ ! -f "${XDG_CONFIG_HOME}/age/chezmoi.txt" ]; then
      mkdir -p "${XDG_CONFIG_HOME}/age"
      logg star '`PRESS ENTER` if you have not set up your encryption token yet'
      age --decrypt --output "${XDG_CONFIG_HOME}/age/chezmoi.txt" "{{ .chezmoi.sourceDir }}/key.txt.age" || EXIT_CODE=$?
      if [ -n "$EXIT_CODE" ]; then
        logg info 'Proceeding without decrypting age encryption key stored at `~/.local/share/chezmoi/home/key.txt.age`'
        logg info 'To have Chezmoi handle your encryption (so you can store your private files publicly) take a look at https://shorturl.at/jkpzG'
        logg info 'Removing all files that begin with encrypted_ because decryption failed'
        find "$HOME/.local/share/chezmoi" -type f -name "encrypted_*" | while read ENCRYPTED_FILE; do
          logg info "Removing $ENCRYPTED_FILE"
          rm -f "$ENCRYPTED_FILE"
        done
      fi
    fi
  fi
fi

### Ensure proper permissions on private key
if [ -f "${XDG_CONFIG_HOME}/age/chezmoi.txt" ]; then
  logg info 'Ensuring proper permissions on Chezmoi / age decryption key'
  logg info 'Chezmoi / age decryption key is stored in '"${XDG_CONFIG_HOME}/age/chezmoi.txt"
  chmod 600 "${XDG_CONFIG_HOME}/age/chezmoi.txt"
fi
```
