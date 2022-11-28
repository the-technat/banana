# Infrastructure

## Control Plane Node (Hetzner)

- SSH-Key which is the default key
- FW rules: only 59245/tcp open, everything else blocked incoming, outgoing has everything open
- Backups enabled (just to be sure)
- Tags (if you want)
- Public IPv4/v6
- Cloud Init:
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
        keyid: 7F92E05B31093BEF5A3C2D38FEEA9169307EA071
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

## Worker Nodes

Not documented here as their setup is different.
