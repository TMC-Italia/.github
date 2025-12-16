# Cloud Project - Hackathon Tasks

**Duration**: 4.5 Hours (with 30 min buffer)  
**Goal**: Automated setup for 3-node cluster with networking, security, distributed storage, and visible management tools

## Pre-Event Checklist

- [ ] 3x Fresh Ubuntu 22.04 LTS PCs (each with 1 SSD)
- [ ] Tailscale pre-auth keys generated
- [ ] Static IPs planned (192.168.1.100-102)
- [ ] tmc-cloud repo cloned with executable scripts

---

## Task 1: Node Setup Script (90 min)

**Goal**: One-command setup script for fresh PC → fully configured node.

**Why**: Eliminate manual setup errors and save time.

**Implementation**:
- Create `scripts/setup-node.sh`
- Install base packages: Docker, git, curl
- Configure static IP via netplan
- Set hostname based on node role
- Install and configure Tailscale with pre-auth key
- Basic system hardening

**Acceptance Criteria**:
- [ ] Single command: `./scripts/setup-node.sh --ip=192.168.1.100 --hostname=tmc-node1`
- [ ] Static IP configured and persists after reboot
- [ ] Hostname set correctly
- [ ] Tailscale connected automatically (no manual auth)
- [ ] DNS resolution works
- [ ] Docker installed and running
- [ ] Script tested on fresh Ubuntu 22.04

---

## Task 2: Security & Firewall (45 min)

**Goal**: Lock down nodes with firewall and SSH hardening.

**Why**: Basic security is essential for production use.

**Implementation**:
- Create `scripts/configure-security.sh`
- Configure UFW firewall:
  - Allow: SSH (22), Tailscale (41641/udp)
  - Storage-specific: Ceph ports (6789, 6800-7300)
- SSH hardening:
  - Disable password auth
  - Key-only authentication
  - Disable root login
- Install fail2ban for SSH protection

**Acceptance Criteria**:
- [ ] UFW enabled with correct rules
- [ ] SSH accessible only with keys
- [ ] fail2ban active and monitoring
- [ ] Can still connect via SSH keys
- [ ] Firewall rules persist after reboot
- [ ] `sudo ufw status` shows all required ports

---

## Task 3: Distributed Storage Investigation & Setup (105 min)

**Goal**: Research and implement distributed storage solution for 3-node cluster.

**Why**: Need shared, fault-tolerant storage across all nodes using available SSDs.

**Phase 1: Research (30 min)**
Evaluate these options for 3 nodes with 1 SSD each:
- **Ceph** (industry standard, complex setup)
- **MicroCeph** (simplified Ceph deployment)
- **GlusterFS** (simpler alternative)
- **Longhorn** (Kubernetes-native)

Document findings: ease of setup, resource requirements, fault tolerance.

**Phase 2: Implementation (75 min)**
Based on research, implement chosen solution:

**Option A: MicroCeph** (Recommended for simplicity)
- Install MicroCeph on all 3 nodes
- Bootstrap cluster on first node
- Add remaining nodes to cluster
- Create storage pool using SSDs
- Configure basic replication

**Option B: GlusterFS** (If Ceph too complex)
- Install GlusterFS on all nodes
- Create replicated volume across 3 nodes
- Mount on all nodes at `/mnt/shared`
- Test replication and failover

**Acceptance Criteria**:
- [ ] Research documented with recommendation
- [ ] Storage solution deployed on all 3 nodes
- [ ] Each node's SSD integrated into cluster
- [ ] Storage accessible from all nodes
- [ ] Can write/read files from any node
- [ ] Data persists if 1 node fails (fault tolerance)
- [ ] Configuration survives reboot
- [ ] Basic performance test completed

---

## Task 4: Visible Management & Monitoring (60 min)

**Goal**: Deploy web-based tools to demonstrate working cluster to management.

**Why**: Management needs to see tangible proof that the infrastructure is operational. Web UIs provide immediate visual confirmation of cluster health, container status, and system metrics.

**Implementation**:

**Phase 1: Portainer Deployment (30 min)**
- Create `scripts/deploy-portainer.sh`
- Deploy Portainer Server on primary node (port 9000)
- Deploy Portainer Agents on remaining nodes (port 9001)
- Use persistent volume on distributed storage
- Configure agent connection to server

**Phase 2: Monitoring Stack (30 min)**
- Create `docker-compose-monitoring.yml` with:
  - **Grafana** (port 3000) - visualization dashboard
  - **Loki** (port 3100) - log aggregation
  - **Promtail** - log collector on each node
- Configure Grafana with Loki data source
- Create basic dashboard showing:
  - Node status (up/down)
  - Container count
  - Recent logs from all nodes

**Acceptance Criteria**:
- [ ] Portainer Server accessible at `http://<node-ip>:9000`
- [ ] All 3 nodes visible in Portainer UI
- [ ] Can view container list across cluster
- [ ] Grafana accessible at `http://<node-ip>:3000`
- [ ] Grafana shows live logs from all nodes via Loki
- [ ] Dashboard displays cluster health metrics
- [ ] Can demonstrate to management: "3 nodes, X containers, live logs"
- [ ] All services use distributed storage (survive node restart)

**Demo Script for Management**:
1. Open Portainer → Show all 3 nodes connected
2. Show running containers across cluster
3. Open Grafana → Show live system logs
4. Restart one node → Show cluster still operational
5. Show logs of the restart event

---

## Overall Success Criteria

**Must Have (Demo-Ready)**:

- [ ] All 3 nodes have static IPs configured
- [ ] Tailscale connected on all nodes
- [ ] SSH hardened (key-only access)
- [ ] Firewall enabled and configured
- [ ] Distributed storage solution chosen and documented
- [ ] Storage accessible from all nodes
- [ ] **Portainer UI showing all 3 nodes**
- [ ] **Grafana + Loki showing live cluster logs**

**Stretch Goals** (if time permits):

- [ ] Automated validation script
- [ ] Storage performance benchmarks
- [ ] Additional Grafana dashboards (CPU, memory, disk)
- [ ] Alerting rules configured

---

## Quick Start

```bash
# 1. Clone repo
git clone https://github.com/TMC-Italia/tmc-cloud
cd tmc-cloud
chmod +x scripts/*.sh

# 2. Set environment
export TAILSCALE_AUTH_KEY=tskey-auth-xxxxx

# 3. Run on each node
./scripts/setup-node.sh --ip=192.168.1.100 --hostname=tmc-node1

# 4. Configure security
./scripts/configure-security.sh

# 5. Setup storage (after research phase)
# Follow chosen solution's setup guide

# 6. Deploy management tools
./scripts/deploy-portainer.sh
docker-compose -f docker-compose-monitoring.yml up -d

# 7. Access UIs
# Portainer: http://192.168.1.100:9000
# Grafana: http://192.168.1.100:3000
```

## Testing Strategy

Use VM snapshots for safe iteration:

1. Create Ubuntu 22.04 VM → take snapshot
2. Run setup scripts
3. If fails → restore snapshot and iterate
4. Test storage failover scenarios
5. Document final configuration
