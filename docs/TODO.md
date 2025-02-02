xattr -d com.apple.quarantine rclone
Create issue about setting up completions - https://github.com/rsteube/lazycomplete

# TODOs

This page outlines various projects and tasks that we are currently working on. Creating a GitHub issue for each of these items would be overkill.

- Add Mamba
- https://containertoolbx.org/install/
- https://github.com/todotxt/todo.txt-cli
- https://github.com/PromtEngineer/localGPT
- https://github.com/StanGirard/quivr
- https://github.com/containers/toolbox
- [IP Fire](https://www.ipfire.org/) - Consider as alternative to pfSense on Qubes.
- `git-credential-manager configure`
- [`git-credential-manager` for WSL](https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/wsl.md)
- Configure Navi to automatically download and use the best cheat repositories
- Google Drive index on Cloudflare https://github.com/menukaonline/goindex-extended
- Go through https://github.com/jaywcjlove/awesome-mac
- https://codesandbox.io/ https://github.com/firecracker-microvm/firecracker
- (https://www.kolide.com/features/checks/mac-firewall)
- Create IP set for CloudFlare [Title](https://firewalld.org/documentation/man-pages/firewalld.ipset.html)
- https://chainner.app/
- https://github.com/kyrolabs/awesome-langchain
- Create seed for Lulu
- https://github.com/essandess/macOS-Fortress
- https://wakatime.com/plugins
- https://github.com/containers/toolbox consider for p10k.zsh file
- Figure out where Vector service fits in
- Figure out if Squid can be used to improve web surfing speed
- https://github.com/mumoshu/variant (With Task)
- https://github.com/marshyski/quick-secure
- https://www.haskell.org/ghcup/install/#how-to-install
- https://github.com/material-shell/material-shell
- https://github.com/arxanas/git-branchless
- https://github.com/mumoshu/variant2
- https://github.com/burnison/tasksync
- https://github.com/Infisical/infisical
- https://github.com/xwmx/nb
- https://github.com/psychic-api/psychic
- https://github.com/pimutils/vdirsyncer
- https://github.com/librevault/librevault

## Upstream

The following items are things we would like to include into the Install Doctor system but are waiting on upstream changes.

- [Actions](https://github.com/sindresorhus/Actions) adds a wide-variety of actions that you can utilize with the macOS Shortcuts app. It is currently only available via the macOS app store. Requested a Homebrew Cask [here](https://github.com/sindresorhus/Actions/issues/127).
- [Color Picker](https://github.com/sindresorhus/System-Color-Picker) is an improved color picker app available on macOS. It is currently only available via the macOS app store. Requested Homebrew Cask [here](https://github.com/sindresorhus/System-Color-Picker/issues/32).
- Consider integrating [LocalAI](https://github.com/go-skynet/LocalAI) which can be used in combination with mods to generate ChatGPT responses locally
- Wait for Homebrew install option for [Warpgate](https://github.com/warp-tech/warpgate)
- Wait for https://github.com/hocus-dev/hocus to get out of alpha for VM management
- Revisit https://github.com/rome/tools when project matures
- Revisit https://github.com/Disassembler0/Win10-Initial-Setup-Script for initial setup of Windows
- Revisit Resilio - seems like they have tools useful for synchronizing VMs
- Consider switching license to [Polyform License Example](https://github.com/dosyago/DiskerNet/blob/fun/LICENSE.md)
- Look into tile managers
- https://github.com/joelbarmettlerUZH/auto-tinder
- https://github.com/hfreire/get-me-a-date
- Keep eye on fig.io for release to Linux and new AI features
- Monitor https://moonrepo.dev/moon as possible mono-repo manager
- Determine whether or not https://webinstall.dev/vim-gui/ will add value to the VIM experience
- Wait for packages to be available for GitHub Actions https://github.com/actions/runner
- Find best Figma plugins here: https://www.figma.com/community/popular

## Review

The following links include software that need to be reviewed before including them into the Install Doctor installer.

### Caddy

- https://authp.github.io/
- https://github.com/caddy-dns/cloudflare
- https://github.com/caddyserver/xcaddy
- https://github.com/luisfarzati/localdots
- https://github.com/mholt/caddy-dynamicdns
- https://github.com/caddyserver/cache-handler
- https://github.com/tailscale/caddy-tailscale
- https://github.com/caddyserver/replace-response
- https://github.com/lindenlab/caddy-s3-proxy
- https://github.com/greenpau/caddy-git
- https://github.com/mholt/caddy-embed
- https://github.com/nathan-osman/caddy-docker

## Docker

The following items are Docker containers that we may want to include as default containers deployed in our system.

- https://github.com/highlight/highlight
- https://github.com/jitsi/jitsi-videobridge
- https://github.com/gitlabhq/gitlabhq
- https://github.com/opf/openproject
- https://github.com/mastodon/mastodon
- https://github.com/huginn/huginn
- https://github.com/chatwoot/chatwoot
- https://github.com/discourse/discourse
- [Title](https://github.com/sipt/shuttle)
- https://github.com/erxes/erxes - CRM
- https://github.com/pawelmalak/flame - Homepage
- https://github.com/thelounge/thelounge - IRC
- https://github.com/vector-im/element-web - Matrix
- https://github.com/outline/outline - Collaborative MD
- https://github.com/nocodb/nocodb - MySQL Spreadsheet
- https://github.com/excalidraw/excalidraw - Hand-drawn Diagrams
- https://github.com/ansible/awx - AWX Ansible Management
- https://github.com/mergestat/mergestat - Git SQL Queries
- https://docs.rundeck.com/docs/administration/install/installing-rundeck.html - Rundeck (Self-Service Desk)
- https://easypanel.io/ - App deployments
- https://www.activepieces.com/docs/install/docker
- https://github.com/activepieces/activepieces - SaaS Automations
- https://github.com/diced/zipline - ShareX / File uploads
- https://github.com/anse-app/anse - ChatGPT interface
- https://github.com/wireapp/wire-webapp - Internal Slack
- https://github.com/jhaals/yopass - OTS web app https://github.com/algolia/sup3rS3cretMes5age
- https://github.com/aschzero/hera - CloudFlare tunnel proxy
- https://supabase.com/ - Firebase alternative
- https://github.com/tiredofit/docker-traefik-cloudflare-companion - Traefik CloudFlare integration
- https://github.com/erxes/erxes - HubSpot alternative
- https://github.com/pawelmalak/flame - Start page
- https://github.com/m1k1o/neko - Docker browser instance
- https://github.com/gristlabs/grist-core - Modern spreadsheet
- https://maddy.email/ / https://github.com/haraka/Haraka
- https://github.com/umputun/remark42 - Comments
- https://github.com/meienberger/runtipi - Home server
- https://github.com/bytebase/bytebase
- https://github.com/IceWhaleTech/CasaOS - Home page https://github.com/ajnart/homarr https://github.com/phntxx/dashboard
- https://github.com/usememos/memos - Memo page
- https://github.com/outline/outline - Team notes
- https://github.com/directus/directus - SQL
- https://github.com/photoprism/photoprism - AI photo manager
- https://github.com/louislam/uptime-kuma - Uptime monitor
- https://github.com/nocodb/nocodb - Airtable alternative
- https://github.com/timvisee/send
- https://github.com/TechnitiumSoftware/DnsServer - DNS proxy server
- https://github.com/lukevella/rallly - Schedule meetings
- https://github.com/chiefonboarding/ChiefOnboarding - Onboarding
- Microserver status page - https://github.com/valeriansaliou/vigil
- https://github.com/pydio/cells - document sharing
- ticket management - https://github.com/Peppermint-Lab/peppermint
- https://github.com/statping-ng/statping-ng
- https://github.com/cortezaproject/corteza - Low-code block workflows
- https://github.com/mirego/accent#-getting-started - Translation tool
- https://github.com/muety/wakapi - Coding time tracking
- https://github.com/subnub/myDrive - Google Drive interface
- https://github.com/Forceu/Gokapi - share files
- https://github.com/gerbera/gerbera - UPnP
- Forward server SSH - https://github.com/warp-tech/warpgate
- https://github.com/hadmean/hadmean - Revisit
- https://spaceb.in/ - Pastebin https://github.com/WantGuns/bin
- https://github.com/AlexSciFier/neonlink - bookmarks
- https://github.com/josdejong/jsoneditor - JSON editor
- https://github.com/AppFlowy-IO/AppFlowy - Notion alternative
- https://github.com/apitable/apitable
- https://github.com/mattermost/mattermost
- https://github.com/duolingo/metasearch
- https://github.com/withspectrum/spectrum
- https://github.com/NginxProxyManager/nginx-proxy-manager
- https://github.com/node-red/node-red
- https://www.overleaf.com/
- https://github.com/caprover/caprover
- [Title](https://github.com/xemle/home-gallery)
- [Title](https://github.com/chartbrew/chartbrew)
- [Title](https://github.com/AlexSciFier/neonlink)
- [Title](https://github.com/ForestAdmin/lumber)
- [Title](https://github.com/subnub/myDrive)
- [Title](https://github.com/mickael-kerjean/filestash)
- [Title](https://github.com/GetStream/Winds)
- [Title](https://github.com/GladysAssistant/Gladys)

## AI

- https://github.com/hwchase17/langchain
- https://github.com/facebookresearch/ImageBind
- https://github.com/nomic-ai/gpt4all

### Kubernetes

The following items may be incorporated into our Kubernetes stack:

- https://github.com/kubevirt/kubevirt
- https://atuin.sh/docs/self-hosting/k8s
- https://github.com/gimlet-io/gimlet
- https://github.com/porter-dev/porter
- https://github.com/spacecloud-io/space-cloud
- https://github.com/meilisearch/meilisearch

## Bare Metal

The projects below are software systems that might be incorporated to handle bare-metal operations or virtual machine management.

- https://theforeman.org/ (VM management)
- https://fogproject.org/ (Backup solution)
- https://github.com/apache/cloudstack (VM management)
- https://www.ovirt.org/ (VM management)
- https://opennebula.io/ (Hybrid-cloud management)
- https://github.com/cloud-hypervisor/cloud-hypervisor (Cloud hypervisor)

## Revisit

The following items have been reviewed but need to be revisited due to complexity or other reasons.

- https://github.com/AmruthPillai/Reactive-Resume
- https://github.com/kubeflow/kubeflow
- https://github.com/leon-ai/leon
- https://github.com/teambit/bit
- https://github.com/Budibase/budibase
- https://github.com/appsmithorg/appsmith
- https://github.com/refined-github/refined-github
- https://github.com/reworkd/AgentGPT
- https://github.com/appwrite/appwrite
- https://github.com/hoppscotch/hoppscotch
- builder.io
- https://github.com/hocus-dev/hocus
- https://github.com/Kanaries/Rath
- cvat.io
- https://github.com/illacloud/illa-builder
- https://github.com/KnowledgeCanvas/knowledge
- https://github.com/siyuan-note/siyuan
- https://github.com/shuttle-hq/shuttle
- https://github.com/open-hand/choerodon
- https://github.com/1backend/1backend
- https://github.com/redkubes/otomi-core
- https://github.com/yunionio/cloudpods
- https://github.com/tkestack/tke
- https://www.rancher.com/
- https://github.com/OpenNebula/one /. https://github.com/OpenNebula/minione
- https://github.com/ConvoyPanel/panel
- https://github.com/hashicorp/nomad
- [Title](https://github.com/Soft/xcolor)
- [Title](https://github.com/Xpra-org/xpra)
- [Title](https://github.com/ksnip/ksnip)
- [Title](https://github.com/leftwm/leftwm)
- [Title](https://github.com/polybar/polybar)
- [Title](https://github.com/kingToolbox/WindTerm)
- [Title](https://github.com/hyprwm/Hypr)
- [Title](https://github.com/Sygil-Dev/sygil-webui)
- [Title](https://github.com/psychic-api/psychic)
- [Title](https://github.com/telekom-security/tpotce)
- [Title](https://flathub.org/apps/com.airtame.Client)
- [Title](https://github.com/Aloxaf/fzf-tab)
  [Title](https://github.com/haproxy/haproxy)
- [Title](https://frappeframework.com/docs/v14/user/en/installation)
- [

](https://github.com/stringer-rss/stringer)

## Bookmarks

- https://cheatsheets.zip/

## Windows

- https://github.com/DDoSolitary/LxRunOffline
