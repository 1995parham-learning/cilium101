# Cilium101

## Introduction

[Cilium](https://cilium.io/) is an open-source networking, security, and observability solution for Kubernetes.
It leverages eBPF (extended Berkeley Packet Filter) to provide high-performance networking and advanced security features
without requiring changes to application code.

This repository provides a working example of setting up Cilium on a local Kubernetes cluster using [Kind](https://kind.sigs.k8s.io/).

## What is Cilium?

### eBPF - The Foundation

Cilium is built on [eBPF](https://ebpf.io/), a revolutionary Linux kernel technology that allows sandboxed programs to run
inside the kernel without changing kernel source code or loading kernel modules. This enables Cilium to:

- Intercept and modify network packets at the kernel level
- Make intelligent routing decisions with minimal overhead
- Collect detailed telemetry without impacting performance

### Key Features

#### Networking

- **CNI Plugin**: Serves as the Container Network Interface for Kubernetes, handling pod-to-pod communication
- **Load Balancing**: Provides highly efficient, eBPF-based load balancing that can replace kube-proxy
- **Multi-Cluster**: Connects and secures workloads across multiple Kubernetes clusters (Cluster Mesh)
- **BGP Support**: Native BGP integration for announcing pod CIDRs and service IPs
- **IPv4/IPv6**: Full dual-stack support

#### Security

- **Network Policies**: Implements Kubernetes NetworkPolicy plus extended CiliumNetworkPolicy with L7 filtering
- **Identity-Based Security**: Security policies based on pod identity rather than IP addresses
- **Transparent Encryption**: WireGuard or IPsec encryption for pod-to-pod traffic
- **Mutual TLS**: Service mesh capabilities with mTLS without sidecars

#### Observability

- **Hubble**: Built-in observability platform providing:
  - Network flow visibility
  - Service dependency maps
  - DNS monitoring
  - HTTP/gRPC metrics
  - Prometheus metrics export

### Architecture

```text
┌────────────────────────────────────────────────────────────┐
│                      User Space                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ Cilium Agent│  │   Hubble    │  │  Cilium Operator    │ │
│  └──────┬──────┘  └──────┬──────┘  └─────────────────────┘ │
├─────────┼────────────────┼─────────────────────────────────┤
│         │    Kernel      │                                 │
│         ▼                ▼                                 │
│  ┌────────────────────────────────────────────────────┐    │
│  │                  eBPF Programs                     │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐            │    │
│  │  │ Routing  │ │ Security │ │  Load    │            │    │
│  │  │          │ │ Policies │ │ Balancer │            │    │
│  │  └──────────┘ └──────────┘ └──────────┘            │    │
│  └────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────┘
```

### Why Cilium over Traditional CNIs?

| Feature           | Traditional CNI (e.g., Flannel) | Cilium        |
| ----------------- | ------------------------------- | ------------- |
| Packet Processing | iptables (user space)           | eBPF (kernel) |
| Performance       | Moderate                        | High          |
| L7 Policies       | No                              | Yes           |
| Observability     | Limited                         | Hubble        |
| Encryption        | External tools                  | Built-in      |
| Identity-aware    | No                              | Yes           |

## Prerequisites

- [Docker](https://www.docker.com/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Cilium CLI](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli)
- [Helm](https://helm.sh/docs/intro/install/) (optional, for Helm-based installation)

## Cluster Setup

The `cluster.yaml` file contains a Kind cluster configuration with:

- 1 control-plane node
- 3 worker nodes
- Default CNI disabled (required for Cilium)

Create the cluster:

```bash
kind create cluster --config cluster.yaml
```

## Installing Cilium

### Using Cilium CLI (Recommended)

```bash
cilium install
```

### Using Helm

```bash
helm repo add cilium https://helm.cilium.io/
helm repo update
helm install cilium cilium/cilium --namespace kube-system
```

## Verification

Check if Cilium is running:

```bash
cilium status --wait
```

Run connectivity tests:

```bash
cilium connectivity test
```

## Cleanup

Delete the cluster:

```bash
kind delete cluster --name cilium101
```

## Resources

- [Cilium Documentation](https://docs.cilium.io/)
- [Cilium Getting Started Guide](https://docs.cilium.io/en/stable/gettingstarted/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
