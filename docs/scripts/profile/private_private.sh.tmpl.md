---
title: Secrets
description: Seperate environment variables file that, when manually sourced, includes secret environment variables
sidebar_label: Secrets
slug: /scripts/profile/private_private.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/dot_config/shell/private_private.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/dot_config/shell/private_private.sh.tmpl
repoLocation: home/dot_config/shell/private_private.sh.tmpl
---
# Secrets

Seperate environment variables file that, when manually sourced, includes secret environment variables

## Overview

This script can be invoked by running `. ~/.config/shell/private.sh` to include secret environment variables
that are populated by Install Doctor during the provisioning process (if they are provided).



## Source Code

```
{{- if (stat (joinPath .host.home ".config" "age" "chezmoi.txt")) -}}
#!/usr/bin/env sh
# @file Secrets
# @brief Seperate environment variables file that, when manually sourced, includes secret environment variables
# @description
#     This script can be invoked by running `. ~/.config/shell/private.sh` to include secret environment variables
#     that are populated by Install Doctor during the provisioning process (if they are provided).

### Ansible
export ANSIBLE_GALAXY_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "ANSIBLE_GALAXY_TOKEN")) }}{{ includeTemplate "secrets/ANSIBLE_GALAXY_TOKEN" | decrypt }}{{ else }}{{ env "ANSIBLE_GALAXY_TOKEN" }}{{ end }}"
export ANSIBLE_VAULT_PASSWORD="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "ANSIBLE_VAULT_PASSWORD")) }}{{ includeTemplate "secrets/ANSIBLE_VAULT_PASSWORD" | decrypt }}{{ else }}{{ env "ANSIBLE_VAULT_PASSWORD" }}{{ end }}"
export AVP="$ANSIBLE_VAULT_PASSWORD"

### Google Cloud SDK
export CLOUDSDK_CORE_PROJECT="{{ .user.gcloud.coreProject }}"
export GCE_SERVICE_ACCOUNT_EMAIL="{{ .user.gcloud.email }}"
export GCE_CREDENTIALS_FILE="$HOME/.config/gcloud/gcp.json"

### CloudFlare
export LEXICON_CLOUDFLARE_TOKEN=""
export LEXICON_CLOUDFLARE_USERNAME="{{ .user.cloudflare.username }}"

### DockerHub
export DOCKERHUB_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "DOCKERHUB_TOKEN")) }}{{ includeTemplate "secrets/DOCKERHUB_TOKEN" | decrypt }}{{ else }}{{ env "DOCKERHUB_TOKEN" }}{{ end }}"
export DOCKERHUB_REGISTRY_PASSWORD="$DOCKERHUB_TOKEN"

### GitHub
export GH_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "GITHUB_TOKEN")) }}{{ includeTemplate "secrets/GITHUB_TOKEN" | decrypt }}{{ else }}{{ env "GITHUB_TOKEN" }}{{ end }}"
export GITHUB_TOKEN="$GH_TOKEN"

### GitLab
export GL_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "GITLAB_TOKEN")) }}{{ includeTemplate "secrets/GITLAB_TOKEN" | decrypt }}{{ else }}{{ env "GITLAB_TOKEN" }}{{ end }}"
export GITLAB_TOKEN="$GL_TOKEN"

### Heroku
export HEROKU_API_KEY="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "HEROKU_API_KEY")) }}{{ includeTemplate "secrets/HEROKU_API_KEY" | decrypt }}{{ else }}{{ env "HEROKU_API_KEY" }}{{ end }}"

### Megabyte Labs
export FULLY_AUTOMATED_TASKS=true

### NPM
export NPM_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "NPM_TOKEN")) }}{{ includeTemplate "secrets/NPM_TOKEN" | decrypt }}{{ else }}{{ env "NPM_TOKEN" }}{{ end }}"

### OpenAI
export OPENAI_API_KEY="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "OPENAI_API_KEY")) }}{{ includeTemplate "secrets/OPENAI_API_KEY" | decrypt }}{{ else }}{{ env "OPENAI_API_KEY" }}{{ end }}"

### PyPi
export PYPI_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "PYPI_TOKEN")) }}{{ includeTemplate "secrets/PYPI_TOKEN" | decrypt }}{{ else }}{{ env "PYPI_TOKEN" }}{{ end }}"

### Snapcraft
export SNAPCRAFT_EMAIL="{{ .user.snapcraft.username }}"
export SNAPCRAFT_MACAROON="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "SNAPCRAFT_MACAROON")) }}{{ includeTemplate "secrets/SNAPCRAFT_MACAROON" | decrypt }}{{ else }}{{ env "SNAPCRAFT_MACAROON" }}{{ end }}"
export SNAPCRAFT_UNBOUND_DISCHARGE="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "SNAPCRAFT_UNBOUND_DISCHARGE")) }}{{ includeTemplate "secrets/SNAPCRAFT_UNBOUND_DISCHARGE" | decrypt }}{{ else }}{{ env "SNAPCRAFT_UNBOUND_DISCHARGE" }}{{ end }}"

### Surge.sh
export SURGE_LOGIN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "SURGE_LOGIN")) }}{{ includeTemplate "secrets/SURGE_LOGIN" | decrypt }}{{ else }}{{ env "SURGE_LOGIN" }}{{ end }}"
export SURGE_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "SURGE_TOKEN")) }}{{ includeTemplate "secrets/SURGE_TOKEN" | decrypt }}{{ else }}{{ env "SURGE_TOKEN" }}{{ end }}"

### Vagrant Cloud
export VAGRANT_CLOUD_TOKEN="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "VAGRANT_CLOUD_TOKEN")) }}{{ includeTemplate "secrets/VAGRANT_CLOUD_TOKEN" | decrypt }}{{ else }}{{ env "VAGRANT_CLOUD_TOKEN" }}{{ end }}"

### Xcodes
# Apple ID username and password
export XCODES_USERNAME="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "APPLE_USERNAME")) }}{{ includeTemplate "secrets/APPLE_USERNAME" | decrypt }}{{ else }}{{ env "APPLE_USERNAME" }}{{ end }}"
export XCODES_PASSWORD="{{ if (stat (joinPath .chezmoi.sourceDir ".chezmoitemplates" "secrets" "APPLE_PASSWORD")) }}{{ includeTemplate "secrets/APPLE_PASSWORD" | decrypt }}{{ else }}{{ env "APPLE_PASSWORD" }}{{ end }}"

{{ end -}}
```
