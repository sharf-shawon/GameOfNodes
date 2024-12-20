# GameOfNodes
# My Hybrid-Cloud Homelab Infrastructure Project
A secure, fully documented hybrid-cloud homelab showcasing on-prem storage, virtualization, Kubernetes (K3s), SSO with Authentik, Cloudflare Zero Trust, and Tailscale overlay networking. This repository includes architecture diagrams, IaC configurations, and best practices for encryption, backup, and performance tuning.

**Author:** Mohammed Sharfuddin Shawon.
**Role:** Architect, Developer, and Administrator.
**Skills Demonstrated:** Systems Administration, Network Engineering, Kubernetes, Cloud Integration, Automation, Security, DevOps Tooling.
**Technologies:** On-Prem Servers (TrueNAS SCALE, Proxmox, Docker), Ubuntu Linux, K3s, Oracle Cloud, Tailscale (Headscale), Authentik, Cloudflare Zero Trust, Terraform, Ansible, Prometheus, Grafana

---

## Table of Contents

1. [Overview](#overview)
2. [Environment Details](#environment-details)
   - [On-Prem Hardware](#on-prem-hardware)
   - [Storage & Backup Strategy](#storage--backup-strategy)
   - [Virtualization & Containers](#virtualization--containers)
   - [Cloud Components](#cloud-components)
3. [Architecture & Network Topology](#architecture--network-topology)
   - [High-Level Architecture](#high-level-architecture)
   - [Network Diagrams](#network-diagrams)
   - [Data Flow Explanation](#data-flow-explanation)
4. [Security Measures](#security-measures)
   - [Identity & Access Management (IAM)](#identity--access-management-iam)
   - [Access & Authentication Flow](#access--authentication-flow)
   - [Encryption & Isolation](#encryption--isolation)
   - [Monitoring & Alerts](#monitoring--alerts)
5. [Performance & Scalability](#performance--scalability)
   - [Load Testing & Benchmarking](#load-testing--benchmarking)
   - [Autoscaling & Capacity Planning](#autoscaling--capacity-planning)
   - [Cost Analysis & Optimization](#cost-analysis--optimization)
6. [Rationale & Design Decisions](#rationale--design-decisions)
   - [Technology Trade-Offs](#technology-trade-offs)
   - [Choice of Kubernetes Distribution](#choice-of-kubernetes-distribution)
   - [Cloud vs. On-Prem Considerations](#cloud-vs-on-prem-considerations)
7. [Reflections & Future Enhancements](#reflections--future-enhancements)
   - [Lessons Learned](#lessons-learned)
   - [Potential Improvements](#potential-improvements)
8. [References & Links](#references--links)

---

## Overview

This project demonstrates a comprehensive hybrid-cloud homelab infrastructure that integrates on-premise servers with public cloud services. The environment is designed for high availability, security, and flexibility, supporting practical implementations of modern IT and DevOps principles. By leveraging TrueNAS SCALE for storage, Proxmox and K3s for virtualization and orchestration, Authentik and Cloudflare Zero Trust for single sign-on (SSO) and identity management, and Tailscale (Headscale) for secure overlay networking, this setup ensures that all data flowing in, out, and within the homelab remains fully encrypted at rest and in transit.

### Key Considerations
Throughout the creation of my hybrid-cloud homelab, I consistently balanced three key considerations. First, I was dedicated to safeguarding my data and securing the home network environment, integrating multiple layers of encryption, authentication, and isolation. Second, as a full-time student, I had to be mindful of cost, opting for budget-friendly, second-hand hardware and low-cost solutions that didn’t compromise on learning potential. Finally, I prioritized energy efficiency, favoring low-power components and careful resource allocation to ensure that ongoing operational costs remained manageable while still meeting my performance and reliability goals.
---

## Environment Details

### On-Prem Hardware

1. **Custom Built TrueNAS SCALE Server (Primary Storage):**  
   - **CPU:** Intel Core i7-6700K  
   - **RAM:** 32GB  
   - **Storage:** 
     - OS: 2x 128GB SATA SSDs (mirrored)  
     - Cache: 2x 1TB NVMe SSDs for caching  
     - Data: 4x 4TB SAS drives for main storage  
   - **Use Cases:** Centralized data storage (NFS), media streaming, automated encrypted backups.

2. **HP EliteDesk 800 G2 & G3 (Docker Servers):**  
   - **G2:** Intel Core i5-6500T, 16GB RAM, 256GB SSD  
   - **G3:** Intel Core i3-8100T, 16GB RAM, 256GB SSD  
   - **Use Cases:** Hosting critical family services (e.g., password manager) behind multiple authentication layers.

3. **Proxmox Cluster (3-Node, Dell Wyse 5070 Thin Clients):**  
   - **Hardware:** Low-powered thin clients repurposed for virtualization.  
   - **Goal:** Evolving into a highly available K3s cluster.  
   - **Use Cases:** Containerized workloads, high availability configurations, and resource scheduling.

### Storage & Backup Strategy

- **Centralized Storage:** All on-prem resources leverage TrueNAS NFS/SCSI shares.  
- **3-2-1 Backup Strategy:**  
  1. **3 copies of data:** Primary (on-prem), remote backup, and cloud backup.  
  2. **2 different media types:** On-prem disks and cloud object storage.
  3. **1 off-site backup:** Geographically distinct cloud storage (US East, Central and West).
- **Encryption:** All backups are encrypted before leaving the homelab, ensuring confidentiality in transit and at rest.

### Virtualization & Containers

- **Docker:** Running lightweight containers on TrueNAS for media streaming and ancillary services.
- **Proxmox & K3s (In Progress):** Virtualizing and orchestrating workloads with an aim towards a fully integrated, scalable container platform.

### Cloud Components

- **Oracle Cloud Always Free Tier VPS:**  
  - Runs Authentik for SSO and Headscale for secure, encrypted overlay networking.
- **Additional VPS for Monitoring & Alerts:**  
  - Centralized monitoring (prometheus/grafana) and Ntfy service for real-time infrastructure notifications.

---

## Architecture & Network Topology

### High-Level Architecture

- **On-Prem:** TrueNAS for storage, EliteDesks for critical services, Proxmox cluster evolving into K3s HA cluster.
- **Cloud:** Oracle Cloud VPS for SSO and Headscale; additional VPS for monitoring multiple sites.
- **Overlay Networking:** Headscale (Tailscale) creates an end-to-end encrypted mesh, linking on-prem and cloud resources securely.
- **SSO & Access Control:** Authentik SSO integrated with Cloudflare Zero Trust ensures centralized authentication and controlled public access.

### Network Diagrams

**On-Prem and Cloud Logical Topology:**

     +----------------------+                          +---------------------------+
     |     Home Network     |                          |  Oracle Cloud VPS (SSO)  |
     | (Local LAN, VLANs)   |                          | Authentik + Headscale    |
     +-----------+----------+                          +------------+-------------+
                 |                                              |
          +------v--------+         +-------------------------+  |
          | TrueNAS SCALE | <------> | HP EliteDesks (G2,G3)   |  |
          |   Storage     | NFS      | Critical Home Services  |  |
          +-------+-------+          +------------+------------+  |
                  |                            |                |
                  |                            |                |
           +------v-------+  Tailscale/         |                |
           |  Proxmox     |  Headscale Overlay  <----------------+
           |  Cluster     | (Future K3s HA)                     
           | (3x Wyse 5070)                                      
           +------^-------+
                  |
                  |
         +--------v--------+         +----------------------------+
         |   Internet      |         | Monitoring VPS (Ntfy)      |
         |    Gateway      | <-------> Multi-Site Monitoring      |
         +--------+--------+          -----------------------------+
                  |
         +--------v---------+
         | Cloudflare Zero  |
         | Trust Proxy & DNS|
         | (Public Services)|
         +------------------+


### Data Flow Explanation

- **Public Services:**  
  - Accessed via Cloudflare-proxied domains, protected by Cloudflare Zero Trust integrated with Authentik SSO.  
  - Users authenticate once to gain access to multiple services, minimizing friction and improving security.
  
- **Private Services (Confidential Data):**  
  - Private DNS resolutions direct clients to private IP addresses accessible only through the Tailscale/Headscale VPN.  
  - All data is encrypted at rest (on-prem and in the cloud) and in transit (TLS, VPN), preventing unauthorized access.

- **Backups & Monitoring:**  
  - On-prem services write data to TrueNAS, which is then encrypted and backed up off-site.  
  - Metrics and logs flow to the monitoring VPS, with Ntfy providing immediate alerts. All inter-site communication is encrypted, ensuring integrity and confidentiality.

---

## Security Measures

### Identity & Access Management (IAM)

- **SSO with Authentik:** Central point of authentication for various services, integrated with Cloudflare Zero Trust to streamline login.
- **Role-Based Access Control (RBAC):** Applied within Kubernetes (as it’s fully implemented) and other services to limit permissions on a need-to-use basis.
- **Least Privilege Principle:** Ensures minimal access necessary for each user, service, or application.

### Access & Authentication Flow

- **Cloudflare Zero Trust + Authentik:**  
  - Publicly accessible services require authentication via Cloudflare Zero Trust, which delegates authentication to Authentik.  
  - Users can leverage single sign-on or one-time codes if they cannot enter credentials (e.g., on a public computer).

- **Private Services via VPN:**  
  - Critical services are not publicly exposed; their DNS points to private IPs accessible only through the Tailscale-based VPN.  
  - No open ports or public IP exposure, and multiple layers of authentication ensure only authorized users gain access.

### Encryption & Isolation

- **Encryption in Transit:**  
  - TLS secures all external traffic.  
  - Tailscale (Headscale) ensures end-to-end encryption between all homelab nodes and cloud services.
  
- **Encryption at Rest:**  
  - All data stored locally (TrueNAS) and remotely (off-site backups) is encrypted.  
  - This includes media, critical services data, configuration files, and backup archives.

- **Network Segmentation:**  
  - VLANs and firewall rules separate management, storage, and DMZ networks on-prem.  
  - Cloudflare firewall rules limit external exposure.  
  - Combined, these measures ensure data confidentiality and integrity across the entire infrastructure.

### Monitoring & Alerts

- **Centralized Logging & Metrics:** Prometheus, Grafana, and Ntfy ensure visibility into system health and performance.
- **Audit Trails:** Authentik, Cloudflare Zero Trust logs, and host-based logging provide forensic traceability.
- **Anomaly Detection:** Suspicious activity triggers alerts via Ntfy, ensuring quick response to potential threats.

---

## Performance & Scalability

### Load Testing & Benchmarking

- **Current State:** Family-critical services running at stable performance; planned load testing as K3s cluster matures.
- **Future Tools:** JMeter or Locust to simulate workloads and assess scaling strategies.

### Autoscaling & Capacity Planning

- **K3s HA (Upcoming):** Supports horizontal scaling of containerized workloads.
- **Proxmox Flexibility:** Quickly adjust VM or container resources as demands change.
- **Hybrid Approach:** On-prem stable resources combined with cloud elasticity to handle spikes and distribute workloads.

### Cost Analysis & Optimization

- **On-Prem Costs:** Hardware investments amortized over time, predictable expenses.
- **Cloud Costs:** Minimal due to free-tier services and low-cost VPS solutions.
- **Selective Cloud Usage:** Employing cloud where it adds value (SSO, VPN, monitoring) while keeping core services in-house.

---

## Rationale & Design Decisions

### Technology Trade-Offs

- **TrueNAS SCALE:** Provides robust storage with enterprise-grade features at homelab scale.
- **Proxmox & K3s:** Flexible virtualization and lightweight Kubernetes for efficient resource utilization.
- **Authentik & Cloudflare Zero Trust:** Seamless SSO integration and global security enforcement.

### Choice of Kubernetes Distribution

- **K3s:** Chosen for its lightweight footprint, ease of setup, and suitability for low-powered hardware while still providing a production-grade Kubernetes environment.

### Cloud vs. On-Prem Considerations

- **On-Prem:** Direct hardware control, low latency, cost efficiency, and secure local storage.
- **Cloud:** Global reach, SSO integration, overlay networking, and minimal overhead for supporting services and backups.

---

## Reflections & Future Enhancements

### Lessons Learned

- **Centralized Auth & Security Layers:** Streamline user experience and ensure robust security posture.
- **Encrypted Everything:** Ensuring that all data is encrypted at rest and in transit prevents unauthorized access and instills confidence in the system’s integrity.
- **Network Simplicity via VPN:** Avoiding port forwarding and public IP exposure reduces attack surface.

### Potential Improvements

- **Multi-Cloud Integration:** Adding another cloud provider for redundancy and vendor independence.
- **Service Mesh:** Deploying Istio or Linkerd for advanced traffic management, observability, and encryption within Kubernetes clusters.
- **Event-Driven Scaling:** Introducing KEDA for granular scaling based on external event metrics.
- **Enhanced Observability:** Implementing distributed tracing (Jaeger) and more sophisticated alerting rules.

---

## References & Links

- **TrueNAS SCALE:** [https://www.truenas.com/truenas-scale/](https://www.truenas.com/truenas-scale/)  
- **Proxmox VE:** [https://www.proxmox.com/en/proxmox-ve](https://www.proxmox.com/en/proxmox-ve)  
- **K3s:** [https://k3s.io/](https://k3s.io/)  
- **Authentik SSO:** [https://goauthentik.io/](https://goauthentik.io/)  
- **Headscale (Tailscale):** [https://github.com/juanfont/headscale](https://github.com/juanfont/headscale)  
- **Cloudflare Zero Trust:** [https://www.cloudflare.com/teams/zero-trust/](https://www.cloudflare.com/teams/zero-trust/)  
- **Prometheus & Grafana:** [https://prometheus.io/](https://prometheus.io/) | [https://grafana.com/](https://grafana.com/)  
- **Ntfy:** [https://ntfy.sh/](https://ntfy.sh/)  
- **Terraform & Ansible:** [https://terraform.io/](https://terraform.io/) | [https://www.ansible.com/](https://www.ansible.com/)

