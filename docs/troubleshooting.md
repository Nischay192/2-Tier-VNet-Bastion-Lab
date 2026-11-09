# Troubleshooting guide

The following are diagnostic procedures relevant to this architecture; they are **not** a claim that every scenario occurred during the lab.

## Backend reports unhealthy

- Check Nginx: `sudo systemctl status nginx`.
- Verify HTTP locally: `curl -i http://127.0.0.1/`.
- Match the load balancer health probe to the listening port and path.
- Review web NSG inbound rules, Azure load balancer probe traffic, and backend pool membership.

## Web page loads but `/api` fails

- Verify the Nginx `location /api` and `proxy_pass` target.
- Confirm application listens on port `8080` and its expected interface.
- Check NSGs and routes between `snet-web` and `snet-app`.
- From the web VM: `curl -i http://<APP_PRIVATE_IP>:8080/`.
- Read Nginx error logs and app logs.

## Bastion connectivity fails

- Check Azure Bastion deployment and the `AzureBastionSubnet`.
- Review applicable NSGs and OS-level access credentials.
- Confirm the target VM is running and reachable in the VNet.

## No outbound internet

- Verify NAT Gateway association with the correct subnet.
- Review NSGs, custom routes, and DNS resolution separately.

## Lesson

A running VM, an open port, a healthy load balancer probe, and a functional API response are **different layers of validation**. Troubleshoot the complete request path.
