# 00 - Basics & Infrastructure

## Introduction

This is the documentation I wrote for my personal K8s cluster.
Why I wrote a documentation about it? Well I'm presumably one of those strange engineers that likes writting docs. Or maybe it's just to that you can learn something from me and I still know how everything has been built?

Anyway, let's get started with some important topics

## Naming

Naming things is hard so I like names that have no relation to IT at all nor follow a specific scheme.

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

For basic principles in my homelab see the [.github](https://github.com/alleaffengaffen/.github) repo.

## Infrastructure

We're building a single K8s cluster.

### Control Plane

I only run a sigle node as my control-plane with local etcd as it's simpler, easier to restore and uses fewer resources (which are expensive for a homelab that's mainly for fun)

This node is currently running on [Hetzner Cloud](https://www.hetzner.com/de/cloud).

There I have created a project `alleaffengaffen` and added the following resources:

- SSH-Key which is the default key
- A server with the following specs:
  - located in Falkenstein
  - type CPX11
  - with backups enabled
  - some tags
  - public IPv4 but no private net nor ipv6!

### Worker Nodes

Here are the minimal requirements for worker nodes:

- public IPv4
- no IPv6
- **no firewall blocking anything**
- ssh access

## Next Step

-> [01 - Operating System](./01_os.md)
