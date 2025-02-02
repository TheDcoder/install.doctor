---
title: NGINX Amplify Join
description: Set up NGINX Amplify and joins the cloud monitoring service dashboard
sidebar_label: 58 NGINX Amplify Join
slug: /scripts/after/run_onchange_after_58-nginx-amplify.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_after_58-nginx-amplify.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_after_58-nginx-amplify.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_after_58-nginx-amplify.sh.tmpl
---
# NGINX Amplify Join

Set up NGINX Amplify and joins the cloud monitoring service dashboard

## Overview

This script installs NGINX Amplify and connects with the user's NGINX Amplify instance, assuming the `NGINX_AMPLIFY_API_KEY`
is defined. NGINX Amplify is a free web application that serves as a way of browsing through metrics of all your connected
NGINX instances.

## Links

* [NGINX Amplify login](https://amplify.nginx.com/login)
* [NGINX Amplify documentation](https://docs.nginx.com/nginx-amplify/#)



## Source Code

```
{{- if or (and (eq .host.distro.family "linux") (stat (joinPath .host.home ".config" "age" "chezmoi.txt")) (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "NGINX_AMPLIFY_API_KEY")) (env "NGINX_AMPLIFY_API_KEY")) -}}
#!/usr/bin/env bash
# @file NGINX Amplify Join
# @brief Set up NGINX Amplify and joins the cloud monitoring service dashboard
# @description
#     This script installs NGINX Amplify and connects with the user's NGINX Amplify instance, assuming the `NGINX_AMPLIFY_API_KEY`
#     is defined. NGINX Amplify is a free web application that serves as a way of browsing through metrics of all your connected
#     NGINX instances.
#
#     ## Links
#
#     * [NGINX Amplify login](https://amplify.nginx.com/login)
#     * [NGINX Amplify documentation](https://docs.nginx.com/nginx-amplify/#)

{{ includeTemplate "universal/profile" }}
{{ includeTemplate "universal/logg" }}

if command -v nginx > /dev/null; then
    logg info 'Downloading the NGINX Amplify installer script'
    TMP="$(mktemp)"
    curl -sSL https://github.com/nginxinc/nginx-amplify-agent/raw/master/packages/install.sh > "$TMP"

    logg info 'Running the NGINX Amplify setup script'
    API_KEY="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "NGINX_AMPLIFY_API_KEY")) }}{{- includeTemplate "secrets/NGINX_AMPLIFY_API_KEY" | decrypt -}}{{ else }}{{- env "NGINX_AMPLIFY_API_KEY" -}}{{ end }}" sh "$TMP"
fi

{{ end -}}
```
