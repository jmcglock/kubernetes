# Security Refactoring - Quick Start Guide

## What Changed?

This security refactoring adds three critical layers of defense:

1. **NetworkPolicies** - Restrict network traffic to only what's needed
2. **RBAC & ServiceAccounts** - Limit Kubernetes API access per application
3. **Pod Security Standards** - Enforce secure pod configurations at namespace level

## Before You Apply

### Prerequisites

- Kubernetes cluster with NetworkPolicy support (Calico, Cilium, or similar CNI)
- Pod Security Admission enabled (Kubernetes 1.23+)
- kubectl access with cluster-admin permissions

### Backup Current State

```bash
# Backup all current resources
kubectl get all --all-namespaces -o yaml > backup-before-security-update.yaml

# Backup specific resources
kubectl get deployments --all-namespaces -o yaml > backup-deployments.yaml
kubectl get networkpolicies --all-namespaces -o yaml > backup-networkpolicies.yaml
kubectl get serviceaccounts --all-namespaces -o yaml > backup-serviceaccounts.yaml
```

## Deployment Order (IMPORTANT!)

Apply changes in this exact order to prevent service disruption:

### Step 1: Update Namespaces (Pod Security Standards)
```bash
# This is safe - only adds labels, doesn't enforce immediately
kubectl apply -f adguard-home/adguard-namespace.yml
kubectl apply -f changedetection/changedetection-namespace.yml
kubectl apply -f firefox/firefox-namespace.yml
kubectl apply -f flame-dashboard/flame-namespace.yml
kubectl apply -f ghost-blog/ghost-namespace.yml
kubectl apply -f homarr/homarr-namespace.yml
kubectl apply -f littlelink/littlelink-namespace.yml
kubectl apply -f mysql/mysql-namespace.yml
kubectl apply -f mysql-workbench/mysql-workbench-namespace.yml
kubectl apply -f nextcloud/namespace.yaml
kubectl apply -f pihole/pihole-namespace.yml
kubectl apply -f smokeping/smokeping-namespace.yml
kubectl apply -f speedtest-tracker/speedtest-tracker-namespace.yml
kubectl apply -f uptime/uptime-namespace.yml

# Or apply all at once:
find . -name "*-namespace.y*ml" -exec kubectl apply -f {} \;
```

### Step 2: Create ServiceAccounts & RBAC
```bash
# Create all ServiceAccounts (must be before deployments reference them)
kubectl apply -f adguard-home/adguard-serviceaccount.yml
kubectl apply -f changedetection/changedetection-serviceaccount.yml
kubectl apply -f firefox/firefox-serviceaccount.yml
kubectl apply -f flame-dashboard/flame-serviceaccount.yml
kubectl apply -f ghost-blog/ghost-serviceaccount.yml
kubectl apply -f homarr/homarr-serviceaccount.yml
kubectl apply -f littlelink/littlelink-serviceaccount.yml
kubectl apply -f mysql/mysql-serviceaccount.yml
kubectl apply -f mysql-workbench/mysql-workbench-serviceaccount.yml
kubectl apply -f nextcloud/nextcloud-serviceaccount.yml
kubectl apply -f pihole/pihole-serviceaccount.yml
kubectl apply -f smokeping/smokeping-serviceaccount.yml
kubectl apply -f speedtest-tracker/speedtest-tracker-serviceaccount.yml
kubectl apply -f teslamate/teslamate-serviceaccount.yml
kubectl apply -f traefik/traefik-serviceaccount.yml
kubectl apply -f uptime/uptime-serviceaccount.yml

# Or apply all at once:
find . -name "*-serviceaccount.yml" -exec kubectl apply -f {} \;
```

### Step 3: Update Deployments (Rolling Update)
```bash
# Update deployments to use new ServiceAccounts
# These will trigger rolling updates
kubectl apply -f adguard-home/adguard-deployment.yml
kubectl apply -f changedetection/changedetection-deployment.yml
kubectl apply -f firefox/firefox-deployment.yml
kubectl apply -f flame-dashboard/flame-deployment.yaml
kubectl apply -f ghost-blog/ghost-deployment.yml
kubectl apply -f homarr/homarr-deployment.yml
kubectl apply -f littlelink/littlelink-deployment.yml
kubectl apply -f mysql/mysql-deployment.yml
kubectl apply -f mysql-workbench/mysql-workbench-deployment.yml
kubectl apply -f nextcloud/app-deployment.yaml
kubectl apply -f nextcloud/db-deployment.yaml
kubectl apply -f pihole/pihole-deployment.yml
kubectl apply -f smokeping/smokeping-deployment.yml
kubectl apply -f uptime/uptime-kuma-deployment.yml

# TeslaMate (apply all components)
kubectl apply -f teslamate/teslamate-deployment.yml
kubectl apply -f teslamate/teslamate-db-deployment.yml
kubectl apply -f teslamate/teslamate-grafana-deployment.yml
kubectl apply -f teslamate/teslamate-mosquitto-deployment.yml

# Watch the rollout
kubectl get pods --all-namespaces -w
```

### Step 4: Apply NetworkPolicies (LAST!)
```bash
# ⚠️  WARNING: This may temporarily disrupt connectivity
# Apply NetworkPolicies last, after verifying pods are running

kubectl apply -f adguard-home/adguard-netpolicy.yml
kubectl apply -f changedetection/changedetection-netpolicy.yml
kubectl apply -f firefox/firefox-netpolicy.yml
kubectl apply -f flame-dashboard/flame-netpolicy.yml
kubectl apply -f ghost-blog/ghost-netpolicy.yml
kubectl apply -f homarr/homarr-netpolicy.yml
kubectl apply -f littlelink/littlelink-netpolicy.yml
kubectl apply -f mysql/mysql-netpolicy.yml
kubectl apply -f mysql-workbench/mysql-workbench-netpolicy.yml
kubectl apply -f nextcloud/nextcloud-netpolicy.yml
kubectl apply -f pihole/pihole-netpolicy.yml
kubectl apply -f teslamate/teslamate-netpolicy.yml
kubectl apply -f traefik/traefik-netpolicy.yml
kubectl apply -f smokeping/smokeping-netpolicy.yml
kubectl apply -f speedtest-tracker/speedtest-tracker-netpolicy.yml
kubectl apply -f uptime/uptime-netpolicy.yml

# Or apply all at once:
find . -name "*-netpolicy.yml" -exec kubectl apply -f {} \;
```

## Verification

### Check Everything is Running
```bash
# Verify all pods are running
kubectl get pods --all-namespaces | grep -v Running

# Should return empty (all pods Running)
```

### Test Application Connectivity
```bash
# Test each application via ingress
curl -k https://your-app.domain.com

# Test internal connectivity (example: Nextcloud to MySQL)
kubectl exec -it $(kubectl get pod -n nextcloud -l app=nextcloud-app -o name) -n nextcloud -- nc -zv nextcloud-db 3306

# Should return: succeeded
```

### Verify Security Controls
```bash
# Check NetworkPolicies exist
kubectl get networkpolicies --all-namespaces

# Check ServiceAccounts are used
kubectl get pods --all-namespaces -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,SERVICEACCOUNT:.spec.serviceAccountName

# Check Pod Security labels
kubectl get namespaces --show-labels | grep pod-security
```

## Troubleshooting

### Application Can't Reach Database

**Symptom**: Application logs show connection timeout to database

**Fix**: Check NetworkPolicy allows connectivity
```bash
# Verify app pod label
kubectl get pod <app-pod> -n <namespace> --show-labels

# Verify database pod label
kubectl get pod <db-pod> -n <namespace> --show-labels

# Check if NetworkPolicy allows this connection
kubectl describe networkpolicy -n <namespace>

# Temporarily test without NetworkPolicy
kubectl delete networkpolicy <policy-name> -n <namespace>
```

### Application Can't Reach Internet

**Symptom**: Application can't download updates, send emails, etc.

**Fix**: Add egress rule to NetworkPolicy
```yaml
egress:
- to:
  - namespaceSelector: {}
  ports:
  - protocol: TCP
    port: 443  # or 80, 25, 587, etc.
```

### Pods Failing to Start (Security Violation)

**Symptom**: Pods fail with Pod Security violation

**Fix**: Check pod events and adjust securityContext
```bash
# Check events
kubectl describe pod <pod-name> -n <namespace>

# Common fixes:
# - Add runAsNonRoot: true
# - Add allowPrivilegeEscalation: false
# - Add capabilities.drop: [ALL]
```

### ServiceAccount Not Found

**Symptom**: Pod fails with "ServiceAccount not found"

**Fix**: Ensure ServiceAccount exists
```bash
# Check if ServiceAccount exists
kubectl get sa -n <namespace>

# Create if missing
kubectl apply -f <namespace>/<app>-serviceaccount.yml

# Restart deployment
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

## Quick Rollback

If you need to rollback everything:

```bash
# 1. Remove NetworkPolicies (restores connectivity)
kubectl delete networkpolicies --all --all-namespaces

# 2. Remove serviceAccountName from deployments
# Edit each deployment and remove the serviceAccountName line
kubectl edit deployment <name> -n <namespace>

# 3. Remove Pod Security labels from namespaces
for ns in adguard changedetection firefox flame ghost homarr littlelink mysql mysql-workbench nextcloud pihole smokeping speedtest-tracker teslamate uptime; do
  kubectl label namespace $ns \
    pod-security.kubernetes.io/enforce- \
    pod-security.kubernetes.io/audit- \
    pod-security.kubernetes.io/warn-
done

# 4. Restore from backup (if needed)
kubectl apply -f backup-before-security-update.yaml
```

## Partial Rollout

You can apply changes to one namespace at a time:

```bash
# Example: Secure only the 'ghost' namespace
kubectl apply -f ghost-blog/ghost-namespace.yml
kubectl apply -f ghost-blog/ghost-serviceaccount.yml
kubectl apply -f ghost-blog/ghost-deployment.yml
kubectl apply -f ghost-blog/ghost-netpolicy.yml

# Verify it works
kubectl get pods -n ghost
curl -k https://blog.yourdomain.com

# Then proceed to next namespace
```

## Testing Checklist

After deployment, test each application:

- [ ] Traefik dashboard accessible
- [ ] AdGuard Home web UI accessible
- [ ] Pi-hole web UI accessible
- [ ] Nextcloud web UI accessible and can access database
- [ ] TeslaMate app, Grafana, and database connectivity
- [ ] Ghost blog accessible
- [ ] Flame dashboard accessible
- [ ] Homarr accessible
- [ ] LittleLink accessible
- [ ] Firefox browser accessible
- [ ] MySQL Workbench accessible and can connect to MySQL
- [ ] Uptime Kuma accessible and can ping targets
- [ ] Smokeping accessible and can ping targets
- [ ] Speedtest Tracker accessible and can run tests
- [ ] ChangeDetection accessible

## Support

For detailed information, see:
- `SECURITY-AUDIT.md` - Security assessment and gaps
- `SECURITY-IMPROVEMENTS.md` - Comprehensive documentation
- `README.md` - General deployment instructions

---

**Last Updated**: 2025-11-17
