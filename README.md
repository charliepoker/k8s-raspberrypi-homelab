# Raspberry Pi Kubernetes Homelab

A complete guide for setting up a 3-node Kubernetes cluster using Raspberry Pi 4 devices for CKA certification preparation and homelab experimentation.

## Hardware Components

- 3x Raspberry Pi 4 8GB RAM with GeeekPi Starter Kit (128GB SD cards)
- 1x Cudy GS108 8-Port Gigabit Ethernet Switch
- 5x Cat6 Ethernet Cables (10ft)
- Power supplies and cases included in starter kits

## Project Structure

```
├── docs/                    # Documentation and guides
├── ansible/                 # Ansible playbooks for automation
├── k8s-manifests/          # Kubernetes YAML files
├── monitoring/             # Monitoring stack configs
├── apps/                   # Sample applications
└── scripts/                # Utility scripts
```

## Quick Start

1. [Hardware Assembly](docs/01-hardware-setup.md)
2. [OS Installation](docs/02-os-installation.md)
3. [Network Configuration](docs/03-network-setup.md)
4. [Kubernetes Installation](docs/04-k8s-installation.md)
5. [Cluster Configuration](docs/05-cluster-config.md)
6. [Sample Applications](docs/06-sample-apps.md)

## Learning Objectives

- CKA certification preparation
- Kubernetes cluster administration
- Container orchestration
- Infrastructure as Code
- Monitoring and logging
- CI/CD pipelines

## Blog Series

This project will be documented as a blog series covering:

- Hardware setup and networking
- Kubernetes installation and configuration
- Application deployment strategies
- Monitoring and observability
- Troubleshooting common issues

---

_Last updated: December 2024_
