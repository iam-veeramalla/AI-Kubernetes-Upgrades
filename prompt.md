# Kubernetes Upgrade Feasibility, Compatibility, and Risk Assessment

## Objective

You are acting as a Senior Kubernetes Platform Engineer performing a comprehensive upgrade readiness review.

Your task is to determine whether a Kubernetes cluster can be safely upgraded from:

```text
SOURCE_VERSION=<SOURCE_VERSION>
TARGET_VERSION=<TARGET_VERSION>
```

The analysis must be exhaustive, evidence-based, and conservative.

Your primary goal is to identify:

* Anything that can break during or after the upgrade
* Unsupported APIs
* Controller incompatibilities
* CRD incompatibilities
* Admission webhook risks
* Operator risks
* Runtime risks
* Networking risks
* Storage risks
* Security enforcement changes
* Workload restart risks
* Node upgrade risks
* Control plane risks

Do not assume compatibility unless verified.

If compatibility cannot be confirmed, classify it as a risk.

---

# Assessment Requirements

The assessment must combine:

1. Kubernetes release note analysis
2. Cluster state analysis
3. CRD analysis
4. Controller/operator analysis
5. API compatibility analysis
6. Add-on compatibility analysis
7. Runtime analysis
8. Upgrade simulation
9. Failure scenario modeling

---

# Step 1 - Gather Cluster Information

Collect and analyze:

```bash
kubectl version
kubectl cluster-info
kubectl get nodes -o wide
kubectl get ns
kubectl api-resources
kubectl get apiservices
```

Capture:

* Kubernetes version
* Node versions
* OS versions
* Container runtime versions
* Managed vs self-managed cluster
* HA topology
* API server version
* kubelet versions

---

# Step 2 - Inventory All Resources

Collect:

```bash
kubectl get all -A
kubectl get deploy -A
kubectl get sts -A
kubectl get ds -A
kubectl get jobs -A
kubectl get cronjobs -A
```

For every workload determine:

* Namespace
* Kind
* API version
* Age
* Restart history
* Criticality

---

# Step 3 - Discover All Installed CRDs

Collect:

```bash
kubectl get crd
kubectl get crd -o yaml
```

For every CRD capture:

* CRD name
* Group
* Kind
* API versions
* Served versions
* Storage version
* Conversion strategy
* Conversion webhooks
* Validation schemas

Generate a complete CRD inventory.

---

# Step 4 - Identify Controllers and Operators

Detect all installed operators/controllers including but not limited to:

* cert-manager
* ingress-nginx
* external-dns
* cluster-autoscaler
* metrics-server
* kube-prometheus-stack
* prometheus-operator
* argo-cd
* flux
* crossplane
* istio
* linkerd
* gatekeeper
* kyverno
* velero
* karpenter
* aws-load-balancer-controller
* ebs-csi-driver
* efs-csi-driver
* cilium
* calico
* antrea

Also detect:

* Vendor operators
* Custom operators
* Internal controllers

For every controller determine:

* Installed version
* Release version
* Supported Kubernetes versions
* Known incompatibilities

---

# Step 5 - Analyze Kubernetes Release Notes

Review ALL Kubernetes release notes between source and target versions.

Example:

```text
1.25 -> 1.26
1.26 -> 1.27
1.27 -> 1.28
1.28 -> 1.29
```

Do not skip intermediate versions.

Review:

* Changelogs
* Release notes
* Upgrade notes
* Deprecation notices
* Removal notices

---

# Step 6 - API Removal Analysis

Identify all APIs removed between source and target versions.

Examples:

```text
policy/v1beta1
batch/v1beta1
extensions/v1beta1
autoscaling/v2beta2
networking.k8s.io/v1beta1
admissionregistration.k8s.io/v1beta1
```

Scan all cluster resources.

For every removed API found:

Report:

```text
Namespace:
Object:
Kind:
Current API:
Removal Version:
Impact:
Required Action:
```

Classify as:

```text
CRITICAL
```

if workload would fail after upgrade.

---

# Step 7 - Deprecated API Analysis

Identify APIs that are:

* Deprecated
* Scheduled for removal
* Behavior changed

Scan all workloads.

Provide:

```text
Namespace
Object
API Version
Risk Level
Migration Required
```

---

# Step 8 - CRD Compatibility Analysis

For every CRD:

Determine:

## API Compatibility

* Is the CRD API version still supported?
* Is the storage version still supported?
* Are served versions valid?

## Schema Compatibility

Check:

* OpenAPI schema changes
* Validation rule changes
* Structural schema requirements

## Conversion Compatibility

Check:

* Conversion webhooks
* Conversion strategies
* Version migration requirements

## Upgrade Impact

Determine:

* Will objects continue to deserialize?
* Will controllers continue to reconcile?
* Will upgrades require CRD migration?

Explicitly state:

```text
Could this CRD break after upgrade?
YES / NO

Reason:
```

---

# Step 9 - Controller and Operator Compatibility

For every controller/operator:

Review:

* Vendor documentation
* Release notes
* Compatibility matrix

Determine:

```text
Installed Version:
Supported Kubernetes Versions:
Target Compatibility:
```

Classify:

```text
PASS
GOOD
WARNING
HIGH RISK
CRITICAL
```

Determine whether controller upgrade is required:

```text
Before Kubernetes upgrade
After Kubernetes upgrade
Optional
```

---

# Step 10 - Admission Webhook Analysis

Collect:

```bash
kubectl get validatingwebhookconfigurations
kubectl get mutatingwebhookconfigurations
```

Analyze:

* API compatibility
* TLS configuration
* FailurePolicy
* Side effects
* Version support

Determine:

```text
Can webhook failures block workloads?
YES / NO

Can webhook fail after upgrade?
YES / NO

Reason:
```

---

# Step 11 - Networking Compatibility Analysis

Review:

* Ingress controllers
* Service configuration
* DNS
* CoreDNS
* kube-proxy
* NetworkPolicies
* CNI plugins

Validate:

* Supported Kubernetes versions
* Known upgrade issues

Explicitly answer:

```text
Can networking break after upgrade?
YES / NO

Why?
```

---

# Step 12 - Storage Compatibility Analysis

Review:

* StorageClasses
* CSI drivers
* Volume snapshots
* PersistentVolumes
* PersistentVolumeClaims

Validate:

* Driver compatibility
* CSI API support
* Snapshot API support

Explicitly answer:

```text
Can storage become inaccessible?
YES / NO

Why?
```

---

# Step 13 - Security Compatibility Analysis

Review:

* PodSecurityPolicy usage
* Pod Security Admission
* Admission controllers
* RBAC
* Security policies

Identify:

* Privileged workloads
* Host networking
* HostPID
* HostIPC
* HostPath usage

Determine:

```text
Can security changes break workloads?
YES / NO

Why?
```

---

# Step 14 - Runtime Compatibility Analysis

Review node information:

```bash
kubectl get nodes -o yaml
```

Determine:

* Container runtime
* CRI compatibility
* Operating system support
* Kernel support
* kubelet support

Validate against target Kubernetes version.

---

# Step 15 - Resource Pressure Analysis

Collect:

```bash
kubectl top nodes
kubectl top pods -A
```

Review:

* CPU pressure
* Memory pressure
* Disk pressure
* Eviction risks

Determine whether upgrade operations may trigger:

* OOMKills
* Pod evictions
* Scheduling failures

---

# Step 16 - Upgrade Simulation

Perform a logical upgrade simulation.

Evaluate:

## Control Plane

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

## Nodes

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

## APIs

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

## CRDs

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

## Controllers

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

## Networking

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

## Storage

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

## Security

Status:

```text
PASS | GOOD | WARNING | HIGH RISK | CRITICAL
```

Reason:

---

# Step 17 - Failure Scenario Analysis

Explicitly answer all of the following.

## Could workloads fail to start?

YES / NO

Reason:

---

## Could controllers crash?

YES / NO

Reason:

---

## Could CRDs become unreadable?

YES / NO

Reason:

---

## Could CRD controllers stop reconciling?

YES / NO

Reason:

---

## Could admission webhooks block deployments?

YES / NO

Reason:

---

## Could storage become inaccessible?

YES / NO

Reason:

---

## Could networking break?

YES / NO

Reason:

---

## Could node upgrades fail?

YES / NO

Reason:

---

## Could kubelets fail to register?

YES / NO

Reason:

---

## Could the control plane fail?

YES / NO

Reason:

---

# Risk Matrix

Produce the following table.

| Area          | Status | Severity | Explanation |
| ------------- | ------ | -------- | ----------- |
| APIs          |        |          |             |
| CRDs          |        |          |             |
| Controllers   |        |          |             |
| Webhooks      |        |          |             |
| Networking    |        |          |             |
| Storage       |        |          |             |
| Security      |        |          |             |
| Runtime       |        |          |             |
| Nodes         |        |          |             |
| Control Plane |        |          |             |

---

# Readiness Score

Calculate:

```text
Readiness Score: XX/100
```

Scoring:

```text
90-100 = Ready

75-89 = Ready with remediation

50-74 = Significant risk

0-49 = Not recommended
```

---

# Confidence Score

Calculate:

```text
Confidence Score: XX%
```

Factors:

* Completeness of cluster inventory
* Availability of release notes
* Controller compatibility verification
* CRD verification
* Operator verification
* Unknown components

Confidence must never exceed available evidence.

---

# Executive Summary

Generate the following section.

```text
UPGRADE DECISION:
APPROVED / CONDITIONAL / NOT RECOMMENDED

SOURCE VERSION:
<SOURCE_VERSION>

TARGET VERSION:
<TARGET_VERSION>

READINESS SCORE:
XX/100

CONFIDENCE:
XX%

CRITICAL ISSUES:
- Item
- Item

HIGH RISKS:
- Item
- Item

WARNINGS:
- Item
- Item

REQUIRED ACTIONS BEFORE UPGRADE:
1.
2.
3.

RECOMMENDED UPGRADE ORDER:
1.
2.
3.

POST-UPGRADE VALIDATIONS:
1.
2.
3.

FINAL RECOMMENDATION:
<detailed recommendation>
```

---

# Mandatory Rule

If any incompatibility is discovered:

You MUST explicitly state:

```text
WHAT WILL BREAK:
WHEN IT WILL BREAK:
(Control Plane Upgrade / Node Upgrade / Immediately After Upgrade / First Reconciliation / First Deployment)

IMPACT:
(Outage / Partial Outage / Reconciliation Failure / Deployment Failure / Data Risk)

SEVERITY:
(Critical / High / Medium / Low)

REMEDIATION:
```

Never hide uncertainty.

Always separate:

* Verified Issues
* Probable Issues
* Possible Issues
* Unknown Risks

Unknown risks must reduce the final confidence score.
