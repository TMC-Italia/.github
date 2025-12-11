# Cloud Project - Hackathon Tasks

**Duration**: 4 Hours  
**Goal**: One script transforms fresh Ubuntu PC → production-ready node

## Pre-Event Checklist

- [ ] Fresh Ubuntu 22.04 LTS on test PC (with VM snapshot)
- [ ] Tailscale pre-auth keys generated
- [ ] Static IPs documented (192.168.1.100-103)
- [ ] tmc-cloud repo cloned with executable scripts

---

## Task 1: Bootstrap Script (90 min)

**Goal**: Create orchestration script for automated node setup.

**Why**: Manual setup is error-prone. Need repeatable automation.

**Implementation**:
- Create `scripts/bootstrap-node.sh`
- Accept `--role=<master|worker|storage>` flag
- Call existing scripts: setup-environment.sh → configure-network.sh
- Basic logging to `/var/log/tmc-cloud/bootstrap.log`
- Simple error handling (exit on failure)

**Acceptance Criteria**:
- [ ] `./scripts/bootstrap-node.sh` completes without errors
- [ ] Supports `--role=<master|worker|storage>` CLI flag
- [ ] Logs to `/var/log/tmc-cloud/bootstrap.log`
- [ ] Shows progress (Step 1/4, 2/4, etc.)
- [ ] Exits with error code on failure
- [ ] Tested on fresh Ubuntu 22.04 VM

---

## Task 2: Security & Firewall (60 min)

**Goal**: Basic firewall and SSH hardening.

**Why**: Prevent unauthorized access.

**Implementation**:
- Create `scripts/configure-security.sh`
- UFW basic rules:
  - Allow: SSH (22), Docker (2376), Portainer (9000)
  - Storage role: Also allow NFS (2049)
- SSH hardening: Disable password auth, require keys only
- Install fail2ban with default SSH jail

**Acceptance Criteria**:
- [ ] UFW enabled with correct ports per role
- [ ] SSH hardened (key-only auth, no root)
- [ ] fail2ban active and monitoring SSH
- [ ] Original configs backed up
- [ ] Can still SSH with key-based auth
- [ ] `sudo ufw status` shows correct rules

---

## Task 3: Network Automation (45 min)

**Goal**: Automate static IP and hostname setup.

**Why**: Manual network config is error-prone.

**Implementation**:

- Enhance `scripts/configure-network.sh`
- Accept IP and hostname as arguments
- Configure netplan for static IP
- Set hostname in /etc/hostname

**Acceptance Criteria**:
- [ ] Static IP configured correctly
- [ ] Hostname set and persists after reboot
- [ ] DNS resolution works (ping google.com)
- [ ] All nodes in `/etc/hosts`
- [ ] Network config survives reboot

---

## Task 4: Storage Setup (30 min)

**Goal**: Configure NFS server/client for shared storage across nodes.

**Why**: Kubernetes needs persistent storage accessible from any node.

**Implementation**:
- Storage node: install NFS server, export `/srv/nfs/storage`
- Other nodes: install NFS client, mount to `/mnt/shared`
- Add fstab entries for persistence
- Set proper permissions (nobody:nogroup)

**Acceptance Criteria**:
- [ ] NFS server running on storage node
- [ ] Clients mount `/mnt/shared` automatically
- [ ] Mounts persist across reboots
- [ ] Can write/read files from any node
- [ ] Minimum 50GB free space verified

---

## Task 5: Portainer Deployment (30 min)

**Goal**: Deploy Portainer for Docker management with web UI.

**Why**: Provides easy-to-use interface for managing containers across cluster.

**Implementation**:
- Create `scripts/deploy-portainer.sh`
- Master: deploy Portainer Server on port 9000
- Workers: deploy Portainer Agent on port 9001
- Use persistent volume for data
- Set restart policy to `always`

**Acceptance Criteria**:
- [ ] Portainer Server accessible at `http://<master-ip>:9000`
- [ ] Admin account created
- [ ] Agents deployed on worker nodes
- [ ] All nodes visible in Portainer UI
- [ ] Can manage containers through web interface

---

## Task 6: Tailscale Integration (30 min)

**Goal**: Auto-connect nodes to Tailscale VPN using pre-auth keys.

**Why**: Manual authentication interrupts automation. Need headless setup.

**Implementation**:
- Enhance `scripts/setup-tailscale.sh`
- Read auth key from `TAILSCALE_AUTH_KEY` environment variable
- Connect with pre-auth: `tailscale up --authkey=$KEY --hostname=tmc-$ROLE`
- Verify connection before continuing

**Acceptance Criteria**:
- [ ] Tailscale connects without user interaction
- [ ] No browser login required
- [ ] Node appears in Tailscale admin console
- [ ] Can ping other Tailscale nodes
- [ ] Connection survives reboot

---

## Task 7: Validation Script (15 min)

**Goal**: Create automated validation to verify all components working.

**Why**: Need quick way to confirm node is properly configured.

**Implementation**:
Create `scripts/validate-node.sh` that checks:
- Docker is running (`docker ps`)
- Portainer accessible (curl localhost:9000)
- Tailscale connected (`tailscale status`)
- Storage mounted and writable
- Firewall enabled with correct rules

**Acceptance Criteria**:
- [ ] Script shows ✓/✗ for each check
- [ ] Clear pass/fail summary
- [ ] Exit code 0 on success, 1 on failure
- [ ] All checks pass on properly configured node

---

## Overall Success Criteria

**Must Have**:
- [ ] Bootstrap script runs on Ubuntu 22.04
- [ ] SSH accessible with keys
- [ ] Docker installed
- [ ] Static IP configured
- [ ] Validation script created

**Stretch Goals** (if time permits):
- [ ] Portainer deployed
- [ ] Tailscale connected
- [ ] NFS storage mounted

---

## Quick Start

```bash
# 1. Prepare
git clone https://github.com/TMC-Italia/tmc-cloud
cd tmc-cloud
chmod +x scripts/*.sh

# 2. Set environment
export TAILSCALE_AUTH_KEY=tskey-auth-xxxxx
export ROLE=master  # or worker, storage

# 3. Bootstrap
./scripts/bootstrap-node.sh

# 4. Validate
./scripts/validate-node.sh
```

## Testing

Use VM snapshots to test iterations:
1. Create Ubuntu 22.04 VM → take snapshot
2. Run bootstrap script
3. If fails → restore snapshot and fix
4. Repeat until validation passes
