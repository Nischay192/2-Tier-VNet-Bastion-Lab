# Azure Two-Tier VNet & Bastion Lab

**Hands-on Azure infrastructure lab demonstrating network segmentation, HTTP load balancing, secure administrative access, and web-to-application connectivity.**

[View architecture and request-flow diagrams](docs/architecture.md)

## Overview

This lab deploys two Ubuntu/Nginx web servers in a dedicated web subnet and an application VM in a separate private subnet. A public **Azure Standard Load Balancer** distributes inbound HTTP traffic between the web servers. Nginx proxies `/api` requests from the web tier to the application VM over the VNet. Azure Bastion provides administrative access without requiring public SSH endpoints on each VM.

**Scope:** An educational two-tier environment—not a production, zone-redundant, zero-downtime architecture.

## Architecture

```mermaid
flowchart TB
    CLIENT(["Internet client"])
    ADMIN(["Administrator"])
    LB["Azure Standard Load Balancer<br/>Public HTTP entry • port 80"]

    subgraph RG["Azure · Canada Central · rg-two-tier-lab"]
        direction TB
        subgraph VNET["vnet-two-tier-lab · 10.10.0.0/16"]
            direction TB
            subgraph WEB["WEB TIER · snet-web · 10.10.1.0/24"]
                direction LR
                W1["vm-web-01<br/>Ubuntu + Nginx<br/>10.10.1.4"]
                W2["vm-web-02<br/>Ubuntu + Nginx<br/>10.10.1.5"]
            end
            subgraph APP["APPLICATION TIER · snet-app · 10.10.2.0/24"]
                API["vm-app-01<br/>Private API · TCP 8080"]
            end
            subgraph BASNET["ADMIN ACCESS · AzureBastionSubnet · 10.10.250.0/26"]
                BAS["Azure Bastion"]
            end
        end
        NAT["NAT Gateway<br/>Outbound connectivity"]
    end

    CLIENT -->|"HTTP"| LB
    LB -->|"Backend pool"| W1
    LB -->|"Backend pool"| W2
    W1 -->|"/api · reverse proxy"| API
    W2 -->|"/api · reverse proxy"| API
    ADMIN -.->|"Secure admin session"| BAS
    BAS -.->|"VM access"| W1
    BAS -.->|"VM access"| W2
    BAS -.->|"VM access"| API
    WEB -.->|"Outbound traffic"| NAT

    classDef external fill:#eaf4ff,stroke:#2563eb,color:#16345d,stroke-width:1.5px;
    classDef entry fill:#dbeafe,stroke:#1d4ed8,color:#173a71,stroke-width:2px;
    classDef web fill:#e7f5ee,stroke:#15803d,color:#14532d,stroke-width:1.5px;
    classDef app fill:#fff1dc,stroke:#d97706,color:#78350f,stroke-width:1.5px;
    classDef admin fill:#ede9fe,stroke:#7c3aed,color:#4c1d95,stroke-width:1.5px;
    classDef egress fill:#f1f5f9,stroke:#64748b,color:#334155;
    class CLIENT,ADMIN external;
    class LB entry;
    class W1,W2 web;
    class API app;
    class BAS admin;
    class NAT egress;
```

**Legend:** Solid arrows = application traffic; dashed arrows = administrative or outbound paths. `nsg-web` and `nsg-app` provide the tier-specific network controls. This is a conceptual diagram of the completed lab, not an Azure export. See [detailed architecture](docs/architecture.md) for network and request-flow views.

## Azure resources

| Component | Lab configuration |
|---|---|
| Resource group | `rg-two-tier-lab` |
| Region | Canada Central |
| Virtual network | `vnet-two-tier-lab` |
| Web subnet | `snet-web` — `10.10.1.0/24` |
| App subnet | `snet-app` — `10.10.2.0/24` |
| Bastion subnet | `AzureBastionSubnet` — `10.10.250.0/26` |
| Web VMs | `vm-web-01` (`10.10.1.4`), `vm-web-02` (`10.10.1.5`) |
| App VM | `vm-app-01` — API on port `8080` |
| Load balancer | `lb-two-tier-lab`, Standard SKU |
| Network security | `nsg-web`, `nsg-app` |
| Other services | Azure Bastion, NAT Gateway |

IP addresses are from the lab environment. The VM, load-balancer and subnet names are included to make the design reproducible; this repository **does not** contain a one-click infrastructure deployment.

## What I validated

- Both web backends displayed **100% healthy** in the Azure Load Balancer backend health view at the time of testing.
- HTTP requests reached the web tier through the public load balancer endpoint.
- Nginx reverse proxy routing on `/api` successfully reached the private application VM.
- The application returned a JSON response containing `"status": "healthy"`.

These observations establish **functional connectivity and probe health**, not tested automatic failover, resilience under load, or an uptime SLO.

## Example request flow

1. A client sends an HTTP request to the load balancer's public IP.
2. The load balancer forwards the request to one of the healthy Nginx web servers.
3. For `/api`, Nginx forwards the request to the application VM on port `8080`.
4. The application returns JSON through the web server to the client.

## Documentation

- [Architecture and network design](docs/architecture.md)
- [Validation checklist and observed results](docs/validation.md)
- [Troubleshooting notes](docs/troubleshooting.md)
- [Sample Nginx reverse-proxy configuration](nginx/reverse-proxy.example.conf)
- [Screenshot guidance](screenshots/README.md)

## Key takeaways

- VM power state, backend probe health, and application-layer behavior are separate checks.
- Subnet boundaries and NSG rules must support the intended traffic flow.
- Bastion separates administrative access from the public application entry point.
- A load balancer and a working API path are building blocks of reliability—not proof of end-to-end high availability.

## Potential improvements

Add infrastructure as code, application logs and metrics, alerting, controlled failure tests, HTTPS, and resilience/load testing. These are **future enhancements**, not implemented or validated features of this lab.

## Cost and security

Azure VMs, Bastion, NAT Gateway, and Standard Load Balancer may incur charges while provisioned. Tear down unused resources and check Azure Cost Management. Do not commit passwords, access tokens, private keys, or connection strings.

**Author:** [Nischay192](https://github.com/Nischay192)
