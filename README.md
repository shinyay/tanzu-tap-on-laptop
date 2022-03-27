# Running Tanzu Application Platform Locally on My Mac

- [Tutorial on Tanzu Developer Center](https://tanzu.vmware.com/developer/guides/tanzu-application-platform-local-devloper-install)

## Description

## Demo

## Features

- feature:1
- feature:2

## Requirement

## Usage

## Installation

### 0. Prerequisite

- Hardware
  - 8 threads
  - 16 GB of RAM
  - 40 GB of free disk space.
- Software
  - Minikube
  - Kubectl
- Account
  - [Docker Hub](https://hub.docker.com/)
  - [Tanzu Network](https://network.tanzu.vmware.com/)

#### Remove Old Tanzu CLI

```shell
rm -rf $HOME/tanzu/cli        # Remove previously downloaded cli files
sudo rm /usr/local/bin/tanzu  # Remove CLI binary (executable)
rm -rf ~/.config/tanzu/       # current location # Remove config directory
rm -rf ~/.tanzu/              # old location # Remove config directory
rm -rf ~/.cache/tanzu         # remove cached catalog.yaml
rm -rf ~/Library/Application\ Support/tanzu-cli/* # Remove plug-ins
```

### 1. Download and Install the Tanzu CLI

Instrall Tanzu CLI

```shell
mkdir $HOME/tanzu
tar -xvf tanzu-framework-darwin-amd64.tar -C $HOME/tanzu
set -x TANZU_CLI_NO_INIT true
cd $HOME/tanzu
sudo install cli/core/v0.11.1/tanzu-core-darwin_amd64  /usr/local/bin/tanzu
```

Check Tanzu CLI Version

```shell
tanzu version
```

```shell
version: v0.11.1
buildDate: 2022-02-14
sha: 4d578570
```

Install Tanzu CLI Plugins

```shell
tanzu plugin install --local cli all
```

```shell
Installing plugin 'package:v0.11.1'
Installing plugin 'secret:v0.11.1'
Installing plugin 'apps:v0.4.1'
Installing plugin 'accelerator:v1.0.1'
Installing plugin 'services:v0.1.2'
✔  successfully installed 'all' plugin
```

## References

## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/34c6fdd50d54aa8e23560c296424aeb61599aa71/LICENSE)

## Author

[shinyay](https://github.com/shinyay)
- twitter: https://twitter.com/yanashin18618
