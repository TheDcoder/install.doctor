---
title: Linux JumpCloud Device Enrollment
description: Enrolls Linux devices as a JumpCloud assets if `JUMPCLOUD_CONNECT_KEY` is defined
sidebar_label: 03 Linux JumpCloud Device Enrollment
slug: /scripts/before/run_onchange_before_03-jumpcloud-linux.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_03-jumpcloud-linux.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_03-jumpcloud-linux.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_03-jumpcloud-linux.sh.tmpl
---
# Linux JumpCloud Device Enrollment

Enrolls Linux devices as a JumpCloud assets if `JUMPCLOUD_CONNECT_KEY` is defined

## Overview

This script enrolls the device as a JumpCloud managed asset. The `JUMPCLOUD_CONNECT_KEY` secret should
be populated using one of the methods described in the [Secrets documentation](https://install.doctor/docs/customization/secrets).

*Note: You should check out the supported systems before trying to enroll devices.*

## JumpCloud on macOS

macOS offers a native device management feature offered through Apple Business. It is the preferred
method since it offers most of the desirable features (like remote wipe). The [JumpCloud MDM documentation](https://support.jumpcloud.com/support/s/article/Getting-Started-MDM)
details the steps required to register macOS MDM profiles with JumpCloud.

## Links

* [JumpCloud device management requirements](https://support.jumpcloud.com/support/s/article/jumpcloud-agent-compatibility-system-requirements-and-impacts1)



## Source Code

```
{{- if and (eq .host.distro.family "linux") (stat (joinPath .host.home ".config" "age" "chezmoi.txt")) (or (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "JUMPCLOUD_CONNECT_KEY")) (env "JUMPCLOUD_CONNECT_KEY")) -}}
#!/usr/bin/env bash
# @file Linux JumpCloud Device Enrollment
# @brief Enrolls Linux devices as a JumpCloud assets if `JUMPCLOUD_CONNECT_KEY` is defined
# @description
#     This script enrolls the device as a JumpCloud managed asset. The `JUMPCLOUD_CONNECT_KEY` secret should
#     be populated using one of the methods described in the [Secrets documentation](https://install.doctor/docs/customization/secrets).
#
#     *Note: You should check out the supported systems before trying to enroll devices.*
#
#     ## JumpCloud on macOS
#
#     macOS offers a native device management feature offered through Apple Business. It is the preferred
#     method since it offers most of the desirable features (like remote wipe). The [JumpCloud MDM documentation](https://support.jumpcloud.com/support/s/article/Getting-Started-MDM)
#     details the steps required to register macOS MDM profiles with JumpCloud.
#
#     ## Links
#
#     * [JumpCloud device management requirements](https://support.jumpcloud.com/support/s/article/jumpcloud-agent-compatibility-system-requirements-and-impacts1)

{{ includeTemplate "universal/profile-before" }}
{{ includeTemplate "universal/logg-before" }}

logg info 'Enrolling device with JumpCloud by running the kickstart script'
curl --tlsv1.2 --silent --show-error --header 'x-connect-key: {{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "JUMPCLOUD_CONNECT_KEY")) }}{{- includeTemplate "secrets/JUMPCLOUD_CONNECT_KEY" | decrypt -}}{{ else }}{{- env "JUMPCLOUD_CONNECT_KEY" -}}{{ end }}' https://kickstart.jumpcloud.com/Kickstart | sudo bash

{{ end -}}```
