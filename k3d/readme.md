# Sources
- k3d: https://k3d.io/stable/#other-installers

# k3d
To test locally you can start up a k3d cluster using Docker

## Prerequisities

- chocholatey
- docker

## Windows Install

- choco install k3d
- k3d cluster create workshop --servers 2 --agents 2 --k3s-arg '"--disable=traefik@server:*"' --k3s-arg '"--flannel-backend=none@server:*"' --k3s-arg '"--disable-network-policy@server:*"' -p "80:80@loadbalancer" -p "443:443@loadbalancer"
- kubectl get nodes

> **Note:** PowerShell may interpret the `*` as a glob character, stripping it from the argument. If you get an error like `invalid format or empty subset in 'server:'`, wrap each `--k3s-arg` value in both single and double quotes as shown above (e.g. `'"--disable=traefik@server:*"'`), or use `cmd.exe` instead.

## Linux Install (RHEL)

- curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
- k3d cluster create workshop --servers 2 --agents 2 --k3s-arg '--disable=traefik@server:*' --k3s-arg '--flannel-backend=none@server:*' --k3s-arg '--disable-network-policy@server:*' -p "80:80@loadbalancer" -p "443:443@loadbalancer"
- kubectl get nodes

> **Note:** Use single quotes around the `--k3s-arg` values to prevent any shell expansion of `*`. If you get `invalid format or empty subset in 'server:'`, ensure the `*` is not being stripped by the shell.

## Cilium install on windows
- curl.exe -L -o cilium.zip https://github.com/cilium/cilium-cli/releases/download/v0.19.2/cilium-windows-amd64.zip
- Expand-Archive cilium.zip -DestinationPath .
- Move-Item cilium.exe C:\Windows\System32\
- Remove-Item cilium.zip
- cilium install
- cilium status --wait

## helm install
- choco install kubernetes-helm
- helm version

# remove k3d cluster
- k3d cluster delete workshop