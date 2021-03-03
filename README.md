# IMPORTANT NOTE

This repository is really out of date, I have moved onto [Flux v2](https://toolkit.fluxcd.io/) and do not use Raspberry Pis anymore. This is now archived and I will leave it up for people to read.

# k3s-gitops-arm

![Kubernetes](https://i.imgur.com/p1RzXjQ.png)

[![Discord](https://img.shields.io/badge/discord-chat-7289DA.svg?maxAge=60&style=flat-square)](https://discord.gg/hk58BZV)

Build a [Kubernetes](https://kubernetes.io/) ([k3s](https://github.com/rancher/k3s)) cluster with RPis and utilize [GitOps](https://www.weave.works/technologies/gitops/) for managing cluster state. I would like to give a shout-out to [k8s-gitops](https://github.com/billimek/k8s-gitops), the big brother of this repo, created by [@billimek](https://github.com/billimek).

This repo uses a lot of multi-arch images provided by [raspbernetes/multi-arch-images](https://github.com/raspbernetes/multi-arch-images).

> **Note**: A lot of files in this project have **@CHANGEME** comments, these are things that are specific to my set up that you may need to change.

* * *

## Prerequisites

### Hardware

- 3x Raspberry Pi 4 (recommended 4GB RAM model)
- 3x SD cards (recommended 32GB)
- 3x USB 3.x flash drives (recommended for local storage)
- A NFS server for storing persistent data (recommended for shared storage)

### Software

> **Note**: I use the fish shell for a lot of my commands. Some will work in Bash but others will not, see [here](docs/fish-shell.md) for more information.

- [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [hypriot/flash](https://github.com/hypriot/flash)
- [alexellis/k3sup](https://github.com/alexellis/k3sup)

* * *

## Directory topology

```bash
.
├── ./ansible        # Ansible playbook to run after the RPis have been flashed
├── ./deployments    # Flux will only scan and deploy from this directory
├── ./setup          # Setup of the cluster
├── ./secrets        # Scripts to generate secrets for Sealed Secrets
└── ./docs           # Documentation
```

* * *

## Network topology

![image](assets/_k3s.png)

|IP|Function|
|---|---|
|192.168.1.1|Router (USG)|
|192.168.1.170|NFS Server|
|192.168.42.1/24|k3s cluster CIDR, VLAN 42|
|192.168.42.23|k3s master (k3s-master)|
|192.168.42.24|k3s worker (k3s-worker-a)|
|192.168.42.25|k3s worker (k3s-worker-b)|

* * *

## Let's get started

### 1. Flash SD Card with Ubuntu

> See [ubuntu.md](docs/ubuntu.md)

### 2. Provision RPis with Ansible

[Ansible](https://www.ansible.com) is a great automation tool and here I am using it to provision the RPis.

> See [ansible.md](docs/ansible.md) and review the files in the [ansible](ansible) folder.

### 3. Install k3s on your RPis using k3sup

[k3sup](https://k3sup.dev) is a neat tool provided by [@alexellis](https://github.com/alexellis) that helps get your k3s cluster up and running quick.

> For manual deployment see [k3sup.md](docs/k3sup.md), and for an automated script see [bootstrap-cluster.sh](setup/bootstrap-cluster.sh)

### 4. Flux and Helm Operator

[Helm](https://v3.helm.sh/) is a package manager for Kubernetes.

[Flux](https://docs.fluxcd.io/en/stable/) is the [GitOps](https://www.weave.works/technologies/gitops/) tool I've chosen to have this Git Repository manage my clusters state.

> For manual deployment see [helm-flux.md](docs/flux-helm-operator.md), and for an automated script see [bootstrap-cluster.sh](setup/bootstrap-cluster.sh)

## Additional Components

### Sealed Secrets

[Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) are a "one-way" encrypted Secret that can be created by anyone, but can only be decrypted by the controller running in the target cluster. The Sealed Secret is safe to share publicly, upload to git repositories, give to the NSA, etc. Once the Sealed Secret is safely uploaded to the target Kubernetes cluster, the sealed secrets controller will decrypt it and recover the original Secret.

> See [sealed-secrets.md](docs/sealed-secrets.md) and review the files in the [secrets](secrets) folder.

### MetalLB

[MetalLB](https://metallb.universe.tf/) is a load-balancer implementation for bare metal Kubernetes clusters, using standard routing protocols.

> Review the file [metallb.yaml](deployments/kube-system/metallb/metallb.yaml)

### Cert Manager

[Cert-Manager](https://github.com/jetstack/cert-manager) will automatically provision and manage TLS certificates in Kubernetes. In this setup I am using Cloudflare as the DNS challenge.

### NGINX Ingress _/engine x/_

[NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) is an Ingress controller that uses ConfigMap to store the NGINX configuration.

> Review the file [nginx-ingress.yaml](deployments/kube-system/nginx-ingress/nginx-ingress.yaml)
