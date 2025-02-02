---
title: Docker Install
description: Ensures Docker is installed, ensures the user can access Docker without sudo, and ensures Docker is configured to use gVisor
sidebar_label: 11 Docker Install
slug: /scripts/before/run_onchange_before_11-install-docker.sh.tmpl
githubLocation: https://github.com/megabyte-labs/install.doctor/blob/master/home/.chezmoiscripts/universal/run_onchange_before_11-install-docker.sh.tmpl
scriptLocation: https://github.com/megabyte-labs/install.doctor/raw/master/home/.chezmoiscripts/universal/run_onchange_before_11-install-docker.sh.tmpl
repoLocation: home/.chezmoiscripts/universal/run_onchange_before_11-install-docker.sh.tmpl
---
# Docker Install

Ensures Docker is installed, ensures the user can access Docker without sudo, and ensures Docker is configured to use gVisor

## Overview

This script ensures Docker is installed and then adds the provisioning user to the `docker` group so that they can
access Docker without `sudo`. It also installs and configures gVisor for use with Docker.

## gVisor

gVisor is included with our Docker setup because it improves the security of Docker. gVisor is an application kernel, written in Go,
that implements a substantial portion of the Linux system call interface. It provides an additional layer of isolation between running
applications and the host operating system. It has gained a lot of attention, perhaps partly, because it is maintained by Google.



## Source Code

```
{{- if ne .host.distro.family "windows" -}}
#!/usr/bin/env bash
# @file Docker Install
# @brief Ensures Docker is installed, ensures the user can access Docker without sudo, and ensures Docker is configured to use gVisor
# @description
#     This script ensures Docker is installed and then adds the provisioning user to the `docker` group so that they can
#     access Docker without `sudo`. It also installs and configures gVisor for use with Docker.
#
#     ## gVisor
#
#     gVisor is included with our Docker setup because it improves the security of Docker. gVisor is an application kernel, written in Go,
#     that implements a substantial portion of the Linux system call interface. It provides an additional layer of isolation between running
#     applications and the host operating system. It has gained a lot of attention, perhaps partly, because it is maintained by Google.

{{ includeTemplate "universal/profile-before" }}
{{ includeTemplate "universal/logg-before" }}

### Install Docker
if [ -d /Applications ] && [ -d /System ]; then
    # macOS
    if [ ! -d /Applications/Docker.app ]; then
        logg info 'Installing Docker on macOS via Homebrew cask'
        brew install --cask docker
    else
        logg info 'Docker appears to be installed already'
    fi
    logg info 'Opening the Docker for Desktop app so that the Docker engine starts running'
    open --background -a Docker
elif command -v apt-get > /dev/null; then
    . /etc/os-release
    if [ "$ID" == 'ubuntu' ]; then
        logg info 'Installing Docker on Ubuntu'
    else
        logg info 'Installing Docker on Debian'
    fi
    sudo apt-get update
    sudo apt-get install -y ca-certificates curl gnupg lsb-release
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL "https://download.docker.com/linux/$ID/gpg" | sudo gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/$ID $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
elif command -v dnf > /dev/null; then
    . /etc/os-release
    if [ "$ID" == 'centos' ]; then
        logg info 'Installing Docker on CentOS'
    elif [ "$ID" == 'fedora' ]; then
        logg info 'Installing Docker on Fedora'
    else
        logg error 'Unknown OS - cannot install Docker' && exit 1
    fi
    sudo dnf -y install dnf-plugins-core
    sudo dnf config-manager --add-repo "https://download.docker.com/linux/$ID/docker-ce.repo"
    sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
elif command -v yum > /dev/null; then
    # CentOS
    logg info 'Installing Docker on CentOS'
    sudo yum install -y yum-utils
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    sudo yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
elif command -v apk > /dev/null; then
    # Alpine
    logg info 'Installing Docker on Alpine'
    sudo apk add --update docker
elif command -v pacman > /dev/null; then
    # Archlinux
    logg info 'Installing Docker on Archlinux'
    sudo pacman -Syu
    sudo pacman -S docker
elif command -v zypper > /dev/null; then
    # OpenSUSE
    logg info 'Installing Docker on OpenSUSE'
    sudo zypper addrepo https://download.docker.com/linux/sles/docker-ce.repo
    sudo zypper install docker-ce docker-ce-cli containerd.io docker-compose-plugin
fi

### Add Docker group on Linux
if command -v groupadd > /dev/null; then
    # Linux
    if ! cat /etc/group | grep docker > /dev/null; then
        logg info 'Creating Docker group'
        sudo groupadd docker
    fi
    logg info 'Adding user to Docker group'
    sudo usermod -aG docker "$USER"
fi

### Boot Docker on start with systemd on Linux machines
if command -v systemctl > /dev/null; then
    # Systemd Linux
    sudo systemctl start docker.service
    sudo systemctl enable docker.service
    sudo systemctl enable containerd.service
fi

### Installs pre-built gVisor using method recommended on official website
function gVisorPreBuilt() {
    logg info 'Installing gVisor using method recommended on official website'
    set -e
    mkdir /tmp/gvisor && cd /tmp/gvisor
    ARCH=$(uname -m)
    URL="https://storage.googleapis.com/gvisor/releases/release/latest/${ARCH}"
    logg info 'Downloading gVisor `runsc` and `containerd-shim-runsc-v1` SHA signatures'
    wget "${URL}/runsc ${URL}/runsc.sha512" "${URL}/containerd-shim-runsc-v1 ${URL}/containerd-shim-runsc-v1.sha512"
    sha512sum -c runsc.sha512 -c containerd-shim-runsc-v1.sha512
    rm -f *.sha512
    chmod a+rx runsc containerd-shim-runsc-v1
    sudo mv runsc containerd-shim-runsc-v1 /usr/local/bin
}

### Installs gVisor using alternate Go method described on the GitHub page
function gVisorGo() {
    # Official build timed out - use Go method
    logg info 'Installing gVisor using the Go fallback method'
    sudo chown -Rf "$(whoami)" /usr/local/src/gvisor
    cd /usr/local/src/gvisor
    echo "module runsc" > go.mod
    GO111MODULE=on go get gvisor.dev/gvisor/runsc@go
    CGO_ENABLED=0 GO111MODULE=on sudo -E go build -o /usr/local/bin/runsc gvisor.dev/gvisor/runsc
    GO111MODULE=on sudo -E go build -o /usr/local/bin/containerd-shim-runsc-v1 gvisor.dev/gvisor/shim
}

### Installs gVisor using the [GitHub developer page method](https://github.com/google/gvisor#installing-from-source). This method requires Docker to be installed
function gVisorSource() {
    ### Ensure sources are cloned / up-to-date
    logg info 'Building gVisor from source'
    if [ -d /usr/local/src/gvisor ]; then
        cd /usr/local/src/gvisor
        sudo git reset --hard HEAD
        sudo git clean -fxd
        sudo git pull origin master
    else
        sudo git clone https://github.com/google/gvisor.git /usr/local/src/gvisor
    fi

    ### Build gVisor
    cd /usr/local/src/gvisor
    sudo mkdir -p bin
    # Wait 5 minutes for build to finish, and if it does not use Go
    # TODO - Generate container-shim-runsc-v1 as well (low priority since this method is not used and is only recommended for development)
    sudo timeout 300 make copy TARGETS=runsc DESTINATION=bin/
    if [ -f ./bin/runsc ]; then
        sudo cp ./bin/runsc /usr/local/bin
    else
        logg error 'Timed out while building `runsc` from source' && exit 6
    fi
}

### Add gVisor
if [ ! -d /Applications ] || [ ! -d /System ]; then
    # Linux
    if ! command -v runsc > /dev/null; then
        # Install gVisor
        gVisorPreBuilt || PRE_BUILT_EXIT_CODE=$?
        if [ -n "$PRE_BUILT_EXIT_CODE" ]; then
            logg warn 'gVisor failed to install using the pre-built method'
            gVisorGo || GO_METHOD_EXIT_CODE=$?
            if [ -n "$GO_METHOD_EXIT_CODE" ]; then
                logg warn 'gVisor failed to install using the Go fallback method'
                gVisorSource || SOURCE_EXIT_CODE=$?
                if [ -n "$SOURCE_EXIT_CODE" ]; then
                    logg error 'All gVisor installation methods failed' && exit 1
                else
                    logg success 'gVisor installed via source'
                fi
            else
                logg success 'gVisor installed via Go fallback method'
            fi
        else
            logg success 'gVisor installed from pre-built Google-provided binaries'
        fi
    else
        logg info '`runsc` is installed'
    fi

    ### Ensure Docker is configured to use runsc
    if [ ! -f /etc/docker/daemon.json ]; then
        # Configure Docker to use gVisor
        # Create /etc/docker/daemon.json
        logg info 'Creating /etc/docker'
        sudo mkdir -p /etc/docker
        if [ -f /usr/local/src/install.doctor/home/dot_config/docker/daemon.json ]; then
            logg info 'Creating /etc/docker/daemon.json'
            sudo cp "/usr/local/src/install.doctor/home/dot_config/docker/daemon.json" /etc/docker/daemon.json
        else
            logg warn '/usr/local/src/install.doctor/home/dot_config/docker/daemon.json is not available so the /etc/docker/daemon.json file cannot be populated'
        fi

        # Restart / enable Docker
        if [[ ! "$(test -d /proc && grep Microsoft /proc/version > /dev/null)" ]] && command -v systemctl > /dev/null; then
            logg info 'Restarting Docker service'
            sudo systemctl restart docker.service
            sudo systemctl restart containerd.service
        fi

        # Test Docker /w runsc
        logg info 'Testing that Docker can load application with `runsc`'
        docker run --rm --runtime=runsc hello-world || RUNSC_EXIT_CODE=$?
        if [ -n "$RUNSC_EXIT_CODE" ]; then
            logg error 'Failed to run the Docker hello-world container with runsc' && exit 5
        else
            logg success 'Docker successfully ran the hello-world container with `runsc`'
        fi
    fi
fi

{{ end -}}
```
