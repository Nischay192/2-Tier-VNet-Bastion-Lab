# Architecture and network design

## Design goal

Demonstrate a segmented two-tier Azure deployment with **one public HTTP entry point**, **two Nginx web VMs**, a **private application VM**, and **separate administrative and outbound access paths**. This is a documented lab, not a production reference architecture.

## 1. Infrastructure at a glance

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

**Reading the diagram:** Solid arrows show application requests; dashed arrows show administrative and outbound connectivity. Network Security Groups (`nsg-web`, `nsg-app`) control the respective subnet traffic; the precise NSG rules are not reproduced here. NAT Gateway is shown as the outbound component without asserting an unverified subnet association.

## 2. Network and security boundaries

```mermaid
flowchart LR
    subgraph PUBLIC["Public boundary"]
        USER(["Internet"])
        LB["Standard Load Balancer<br/>Public HTTP frontend"]
    end
    subgraph VNET["Azure Virtual Network · 10.10.0.0/16"]
        subgraph WEB["snet-web · 10.10.1.0/24 · nsg-web"]
            W1["vm-web-01<br/>10.10.1.4"]
            W2["vm-web-02<br/>10.10.1.5"]
        end
        subgraph APP["snet-app · 10.10.2.0/24 · nsg-app"]
            A["vm-app-01<br/>API · :8080"]
        end
        subgraph B["AzureBastionSubnet · 10.10.250.0/26"]
            BST["Azure Bastion"]
        end
    end
    OPS(["Administrator"])
    USER -->|"HTTP"| LB
    LB --> W1
    LB --> W2
    W1 -->|"Internal /api"| A
    W2 -->|"Internal /api"| A
    OPS -.-> BST
    BST -.-> W1
    BST -.-> W2
    BST -.-> A
    classDef public fill:#dbeafe,stroke:#2563eb,color:#1e3a8a;
    classDef web fill:#dcfce7,stroke:#15803d,color:#14532d;
    classDef app fill:#ffedd5,stroke:#ea580c,color:#7c2d12;
    classDef ops fill:#ede9fe,stroke:#7c3aed,color:#4c1d95;
    class USER,LB public;
    class W1,W2 web;
    class A app;
    class OPS,BST ops;
```

| Boundary | Service / configuration | Role |
|---|---|---|
| Public endpoint | Standard Load Balancer | Distributes inbound HTTP across the web backends |
| Web subnet | `snet-web` (`10.10.1.0/24`), `nsg-web` | Hosts the two Ubuntu/Nginx VMs |
| App subnet | `snet-app` (`10.10.2.0/24`), `nsg-app` | Hosts the private API VM on port `8080` |
| Administration | Azure Bastion, `AzureBastionSubnet` (`10.10.250.0/26`) | VM access without exposing per-VM public SSH endpoints |
| Egress | NAT Gateway | Outbound network connectivity |

The lab uses `rg-two-tier-lab` in **Canada Central** and `vnet-two-tier-lab` (documented as `10.10.0.0/16`). Recorded web VM addresses were `10.10.1.4` and `10.10.1.5`. The app VM's private IP is not included because it was not captured in the available notes.

## 3. End-to-end request flow

```mermaid
sequenceDiagram
    autonumber
    actor Client
    participant LB as Azure Load Balancer
    participant Web as Selected Nginx web VM
    participant API as Private app VM
    Client->>LB: HTTP GET /api
    Note over LB,Web: Healthy web backend selected by LB
    LB->>Web: Forward HTTP request
    Web->>API: Reverse proxy to TCP 8080
    API-->>Web: JSON response (status: healthy)
    Web-->>LB: HTTP response
    LB-->>Client: HTTP response
```

**Validated during the lab:** Both load balancer backends displayed 100% healthy, HTTP reached the web tier, and `/api` returned a successful response from the private application tier. See [validation](validation.md).

## 4. Security and operational notes

- The web tier is the public application entry path; the app tier is reached internally through the web VMs.
- Two separate subnets and NSGs make tier-to-tier dependencies explicit.
- Azure Bastion is the administration path; NAT Gateway is an outbound traffic component, not a replacement for inbound load balancing.
- Nginx proxy configuration and the application listener must agree on the internal port and URL path.
- The diagram is conceptual. Exact NSG rule values and NAT Gateway subnet associations should be confirmed against screenshots or exports before adding them here.

## 5. Reliability limitations

The web tier contains **two** VMs, but the application tier contains **one** VM and remains a single point of failure. This lab verified connectivity and backend probe health. It did **not** establish zone redundancy, automated recovery, failover under fault injection, performance at scale, or a production uptime guarantee.
