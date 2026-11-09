# Screenshot evidence guide

This folder holds **actual screenshots from the Azure Two-Tier VNet & Bastion Lab**. The names below are a suggested organization checklist, **not proof that each screenshot was captured or uploaded**. Keep the original images; rename them only when uploading, or I'll match their actual filenames after upload.

## 1. Azure infrastructure and network

| Suggested file | What it should show |
|---|---|
| `01-resource-overview.png` | Resource group and deployed Azure services |
| `02-vnet-address-space.png` | `vnet-two-tier-lab` address space |
| `03-vnet-subnets.png` | `snet-web`, `snet-app` and `AzureBastionSubnet` |
| `04-web-vms.png` | Both Ubuntu web VMs and their private addressing |
| `05-app-vm.png` | Private application VM configuration |

## 2. Network security and administration

| Suggested file | What it should show |
|---|---|
| `06-nsg-web-rules.png` | Web-tier NSG rules (HTTP, load-balancer probe, Bastion access) |
| `07-nsg-app-rules.png` | App-tier NSG rules for web-to-app traffic on TCP 8080 and admin access |
| `08-bastion-overview.png` | Azure Bastion deployment/connection view without sensitive login details |
| `09-nat-gateway.png` | NAT Gateway and its relevant configuration/subnet association |

## 3. Load balancing and application validation

| Suggested file | What it should show |
|---|---|
| `10-load-balancer-overview.png` | Standard Load Balancer frontend and backend configuration |
| `11-load-balancer-rules-probe.png` | Load-balancing rule and HTTP health probe |
| `12-load-balancer-backend-health.png` | Both web backends reported healthy |
| `13-web-response.png` | Successful web-tier HTTP response through the load balancer |
| `14-api-response.png` | `/api` JSON response originating from the private application tier |
| `15-nginx-reverse-proxy.png` | Nginx proxy configuration or successful service check, if captured |

## Optional evidence — only if actually tested

- A controlled web-VM failure and continued HTTP response
- Confirmation that the app tier is not directly accessible from the public internet
- CLI connectivity and troubleshooting commands

The README should highlight the most meaningful **4–6 screenshots**; additional screenshots can remain in this folder or be linked from the detailed documentation. This avoids turning the project homepage into a long image gallery.

## Security and integrity

Before publishing, redact passwords, tokens, subscription/tenant identifiers, account names, sensitive session details, and anything else private. Screenshots should support the exact outcomes observed; do not add fabricated or placeholder test evidence.
