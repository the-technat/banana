# 00 - Basics & Infrastructure

## Introduction

This is the documentation I wrote for my personal K8s cluster.
Why I wrote a documentation about it? Well I'm presumably one of those strange engineers that likes writting docs. Or maybe it's just to that you can learn something from me and I still know how everything has been built?

Anyway, let's get started with some important topics

## Naming

Naming thins is hard so I like names that have no relation to IT at all nor follow a specific scheme.

I name:

- my homelab as a hole `alleaffengaffen`
- my k8s cluster `banana`
- my private domain used inside K8s `banana.k8s`
- my public domain used to expose things `alleaffengaffen.ch`
- my mail for accounts related to the homelab `banane@alleaffengaffen.ch`
- anything else must have either a fruit or animal in it's name:
  - e.g master nodes are `hawk`'s
  - worker nodes are `minion`'s

## Basics

Here are some general principles I follow:

- All my code is stored in an Github organization called `alleaffengaffen`.
- The issues in the `banana` repo serve as my bucket list of things I wanted to try out
- My private network is not at home (as a homelab would assume) but in my tailnet `alleaffengaffen.org.github` using an awesome mesh-to-mesh VPN called [tailscale](https://tailscale.com).
- My IDP where possible is Github.
- I'm not using a specific cloud provider as my tailnet is cloud-agnostic.
- I prefer Ubuntu 22.04 as my OS for servers
- I don't automate stuff as I want to learn something, things can be automated when it must be efficient which my homelab is not
- But I use GitOps and K8s operators
- I'm never going to host real productive services on my homelab. But services that aren't really productive can be hosted there (like the website `alleaffengaffen.ch` or a private photo gallery only having a bit of traffic)
- HA is not a requirement and mustn't be considered at any time
- Use plain tools and do stuff on your own where possible, to learn something (e.g use kubeadm, not k3s or any other fancy tool)
- Harden everything from the beginning!

## Infrastructure

We're building a single K8s cluster.

### Tailnet config

My tailnet is configured using the [policy.hujson](./../policy.hujson) according to [GitOps for tailscale ACLs](https://tailscale.com/kb/1204/gitops-acls/).

Note: Since Hetzner injects their own DNS servers onto every server and don't remove the IPv6 ones if you don't have an IPv6, we need to override local DNS in the tailscale to make sure coredns has valid nameservers configured.

### Control Plane

I only a sigle node as my control-plane with local etcd as it's simpler, easier to restore and uses fewer resources (which are expensive for a homelab that's mainly for fun)

This node is currently running on [Hetzner Cloud](https://www.hetzner.com/de/cloud).

There I have created a project `alleaffengaffen` and added the following resources:

- SSH-Key which is the default key
- A FW that only allows tcp/59245 from the internet but open everything for outgoing traffic
- A server with the following specs:
  - located in falkenstein
  - type CPX11
  - with backups enabled
  - some tagsv
  - public IPv4/v6 but no private net
  - firewall applied
  - a cloud-init config:

  ```yaml
  #cloud-config <hostname>

  locale: en_US.UTF-8
  timezone: UTC
  users:
    - name: technat
      groups: sudo
      sudo: ALL=(ALL) NOPASSWD:ALL # Allow any operations using sudo
      gecos: "Admin user created by cloud-init"
      shell: /usr/bin/bash
      ssh-authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJov21J2pGxwKIhTNPHjEkDy90U8VJBMiAodc2svmnFC cardno:18 055 612

  apt:
    sources:
      kubernetes:
        source: "deb [signed-by=$KEY_FILE] https://apt.kubernetes.io/ kubernetes-xenial main"
        keyid: B53DC80D13EDEF05
  package_update: true
  package_upgrade: true
  packages:
  - vim
  - git
  - wget
  - curl
  - dnsutils
  - containerd
  - apt-transport-https
  - ca-certificates
  - kubeadm
  - kubectl
  - kubelet

  write_files:
  - path: /etc/modules-load.d/containerd.conf
    content: |
      overlay
      br_netfilter
  - path: /etc/sysctl.d/99-kubernetes-cri.conf
    content: |
      net.bridge.bridge-nf-call-iptables  = 1
      net.ipv4.ip_forward                 = 1
      net.bridge.bridge-nf-call-ip6tables = 1
  - path: /etc/ssh/sshd_config
    content: |
      Port 59245
      PermitRootLogin no
      PermitEmptyPasswords no
      PasswordAuthentication no
      PubkeyAuthentication yes
      ChallengeResponseAuthentication no
      KbdInteractiveAuthentication no
      UsePAM yes
      X11Forwarding no
      PrintMotd no
      AcceptEnv LANG LC_*
      Subsystem    sftp    /usr/lib/openssh/sftp-server
      Include /etc/ssh/sshd_config.d/*.conf
  runcmd:
    - sudo apt-mark hold kubelet kubeadm kubectl
    - sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX="systemd.unified.cgroup_hierarchy=1"/g' /etc/default/grub
    - sudo update-grub
    - sudo mkdir -p /etc/containerd
    - sudo containerd config default | sudo tee -a /etc/containerd/config.toml
    - sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

  power_state:
    mode: reboot
    timeout: 30
    condition: true
  ```

### Worker Nodes

Some of them are at home, some of them are in a cloud, I don't try to keep a list of all of them.

## Next Step

-> [01 - Operating System](./01_os.md)
