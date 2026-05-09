## RCA Analysis #4: dev1/foundation Availability Drop

**Document**: RCA for frontend pod deployment failure due to node provisioning issues

### Incident Overview
- **PRB ID**: Not specified
- **Severity**: Not explicitly stated (inferred P1 - availability impact)
- **Type**: Platform integration failure (Karpenter + Kubernetes scheduling)
- **Date**: Not specified in document
- **Status**: Running analysis document (investigation ongoing, less structured than typical RCA)

### Timeline Summary
| Phase | Timestamp | Duration from Start | Notes |
|-------|-----------|---------------------|-------|
| Deployment Initiated | Unknown | 0 | Frontend deployment triggered |
| Pods Enter Pending State | Unknown | Unknown | TSC prevents scheduling on existing nodes |
| Karpenter Attempts Provisioning | Unknown | Unknown | New nodes requested due to unschedulable pods |
| Node Join Failures | Unknown | Unknown | Kubelet registration failures on new nodes |
| Detection | Unknown | **TTD: Unknown** | Not documented when issue was noticed |
| Root Cause Identified | Unknown | **TTD: Unknown** | TSC + node join failure interaction discovered |
| Resolution | Unknown | **TTR: Unknown** | Resolution method not documented |

**Note on Timeline**: This RCA lacks structured timeline data, making it impossible to calculate TTD/TTR. The absence of timestamps is itself a finding - incident documentation quality varies significantly.

**What information is missing**:
- When did deployment start?
- When were pending pods first noticed?
- How long did pods stay in Pending state?
- When did investigation begin?
- How was the issue eventually resolved?
- What was customer/service impact duration?

### Critical Delays Analysis

#### Detection Phase (duration unknown)

**What likely delayed detection**:
1. **No alerting on pods stuck in Pending state**: K8s deployments can have brief pending periods during normal operations - unclear if monitoring distinguishes "normal pending" from "stuck pending"
2. **No alerting on Karpenter node join failures**: New nodes were requested but failed to join cluster - this critical signal may not have been monitored
3. **No deployment progress monitoring**: Deployment rollout stalled, but unclear if system alerted on "deployment not progressing for X minutes"
4. **No TSC violation alerting**: Topology spread constraints blocked scheduling, but no alert on "pods cannot be scheduled due to TSC"

**Key question for gap analysis**: 
- Was this detected by automated monitoring or by human observation (e.g., someone checking deployment status)?
- If automated: How long between pod Pending and alert firing?
- If manual: Why was someone looking? Was it triggered by customer report?

**What should have alerted**:
- Alert: "Deployment rollout stalled for >5 minutes (0% progress)"
- Alert: "Pods in Pending state for >5 minutes with reason: TopologySpreadConstraint"
- Alert: "Karpenter node provisioning failed: kubelet registration timeout"
- Alert: "Node join failure rate >50% in last 10 minutes"

#### Diagnosis Phase (duration unknown)

**What made diagnosis complex**:
1. **Multi-layer failure**: Problem required understanding interaction between:
   - Kubernetes scheduling (TSC preventing pod placement)
   - Karpenter provisioning (new node creation)
   - Node bootstrapping (kubelet registration)
   - Deployment rollout logic (stuck waiting)
2. **Hidden failure**: Kubelet registration failures may not be immediately visible - requires checking:
   - Karpenter logs (did provisioning succeed?)
   - Node status (is node Ready?)
   - Kubelet logs (why didn't it join?)
3. **TSC as a constraint**: Topology spread constraints are often misconfigured - requires understanding:
   - What is the TSC policy? (maxSkew: 2, DoNotSchedule)
   - What is current topology distribution?
   - Why can't existing nodes satisfy the constraint?

**Manual investigation steps likely required**:
1. Check deployment status: `kubectl rollout status deployment/frontend`
2. Check pod status: `kubectl get pods -l app=frontend` → see Pending
3. Check pod events: `kubectl describe pod <pod-name>` → see "TopologySpreadConstraint"
4. Check TSC policy: `kubectl get deployment/frontend -o yaml` → see maxSkew: 2
5. Check node count by zone: `kubectl get nodes --label-columns topology.kubernetes.io/zone`
6. Check Karpenter logs: Are new nodes being created?
7. Check node status: Are new nodes joining cluster?
8. Check kubelet logs on failed nodes: Why registration failed?

**Key question**: How long did this investigation take? Each step requires knowledge and manual action.

#### Resolution Phase (method unknown)

**Possible resolution approaches**:
1. **TSC softening**: Change TSC from `DoNotSchedule` to `ScheduleAnyway` (allows violations if necessary)
2. **Manual node provisioning**: Manually add nodes to correct zones to satisfy TSC
3. **Fix node join issue**: Resolve kubelet registration failure (platform fix)
4. **Disable TSC temporarily**: Remove TSC from deployment spec to unblock rollout
5. **Rollback deployment**: If new deployment introduced the TSC issue, rollback

**Unknown**:
- Which approach was used?
- How long did resolution take?
- Was rollback considered/attempted?
- What was impact on service availability?

### Automation Assessment

#### High-Value Automation Opportunities

1. **Deployment Rollout Stall Detection**
   - **Phase**: Detection
   - **Impact**: Reduces TTD from "unknown" to <5 minutes
   - **Description**: Alert when deployment rollout shows 0% progress for >5 minutes
   - **Feasibility**: High - K8s deployment status is exposed via API
   - **Implementation**: Monitor `kubectl rollout status` or deployment conditions: `AvailableReplicas < DesiredReplicas` sustained for threshold duration

2. **Pod Pending Reason Analysis**
   - **Phase**: Detection + Initial Diagnosis
   - **Impact**: Provides immediate context for why pods stuck
   - **Description**: When pods enter Pending state, extract reason from events:
     - "TopologySpreadConstraint" → TSC blocking scheduling
     - "Insufficient CPU/memory" → Resource exhaustion
     - "Node selector doesn't match" → Configuration error
   - **Feasibility**: High - pod events contain structured reason
   - **Implementation**: Parse `kubectl describe pod` events, alert with specific reason
   - **Value**: Reduces diagnosis from "why are pods pending?" to "TSC is blocking, check distribution"

3. **Karpenter Node Join Failure Alerting**
   - **Phase**: Detection
   - **Impact**: Surfaces hidden layer of failure (nodes provisioned but not joining)
   - **Description**: Alert when:
     - Karpenter provisions node
     - Node doesn't reach Ready state within 5 minutes
     - Kubelet registration errors detected in node/Karpenter logs
   - **Feasibility**: Medium - requires access to Karpenter controller logs + node status
   - **Implementation**: Correlate Karpenter provisioning events with node Ready status
   - **Value**: Makes invisible failure visible - "autoscaling is broken"

4. **TSC Configuration Validation**
   - **Phase**: Prevention (pre-deployment)
   - **Impact**: Prevents incident entirely if TSC misconfigured
   - **Description**: When deployment has TSC with `DoNotSchedule`:
     - Validate current topology distribution can satisfy constraint
     - Warn if maxSkew too strict (e.g., maxSkew:1 in unbalanced cluster)
     - Suggest `ScheduleAnyway` if distribution can't be satisfied
   - **Feasibility**: Medium - requires understanding cluster topology and TSC semantics
   - **Implementation**: Pre-deployment validation hook or admission controller
   - **Value**: Catches configuration errors before they cause production impact

#### Medium-Value Opportunities

1. **Deployment Dependency Health Check**
   - **Phase**: Diagnosis
   - **Impact**: Speeds up diagnosis by checking all dependencies
   - **Description**: When deployment stalls, automatically check:
     - Cluster autoscaler (Karpenter) health
     - Node health across zones
     - Network/control plane health
     - Recent configuration changes (TSC added?)
   - **Feasibility**: High - data exists, needs aggregation
   - **Benefit**: Provides full context in single view vs. manual checks

2. **Automated TSC Softening Suggestion**
   - **Phase**: Diagnosis → Resolution
   - **Impact**: Reduces time to resolution by suggesting fix
   - **Description**: When TSC blocks scheduling + node provisioning failing:
     - Suggest: "Change TSC to ScheduleAnyway to unblock deployment"
     - Provide command: `kubectl patch deployment/frontend -p '...'`
     - Explain trade-off: "Allows uneven distribution temporarily"
   - **Feasibility**: Medium - requires understanding safe resolution paths
   - **Benefit**: Reduces "what do I do?" uncertainty during incident

3. **Node Bootstrap Failure Diagnosis Runbook**
   - **Phase**: Diagnosis
   - **Impact**: Guides investigation of node join failures
   - **Description**: When kubelet registration fails, provide checklist:
     - Check node IAM role permissions
     - Check network connectivity to K8s API
     - Check kubelet logs for specific error
     - Check if cluster CA cert rotated
     - Link to relevant platform team runbook
   - **Feasibility**: High - codify existing troubleshooting knowledge
   - **Benefit**: Reduces expertise dependency (junior engineers can follow)

#### Low-Value / Not Automatable

1. **Platform-Level Node Join Fixes**
   - **Why not automatable (for application team)**: Node join failures are often caused by:
     - Infrastructure/platform configuration (IAM, networking, AMI issues)
     - K8s control plane issues
     - AWS API throttling/issues
   - **Why low automation value**: Requires platform team expertise and access
   - **Resolution**: Platform team needs to fix underlying issue (not something app team can auto-remediate)

2. **TSC Policy Design Decisions**
   - **Why not automatable**: Deciding correct TSC policy requires:
     - Understanding availability requirements (is zone distribution critical?)
     - Understanding deployment patterns (rolling update, blue/green?)
     - Business trade-offs (availability vs. deployment speed)
   - **Human decision needed**: Architect/SRE must decide appropriate constraints

3. **Rollback vs. Forward-Fix Decision**
   - **Why not automatable**: Requires judgment:
     - Is this a new deployment issue or platform issue?
     - Is rollback safe? (data migration concerns?)
     - Is forward fix (TSC softening) acceptable?
     - What's impact of each option?
   - **Human decision needed**: Incident commander must weigh options

#### Special Considerations

**Platform Integration Failure Pattern**:
- This incident is caused by interaction between multiple platform components:
  - K8s scheduler (enforcing TSC)
  - Karpenter autoscaler (provisioning nodes)
  - Node bootstrap process (kubelet registration)
- Failures at lower layers (node join) manifest as symptoms at higher layers (pods pending)
- Diagnosis requires understanding the full stack

**Automation challenges**:
1. **Hidden failures**: Node join failures not visible in pod/deployment status
2. **Multi-layer correlation**: Need to connect "pods pending" → "TSC blocking" → "nodes not joining"
3. **Platform vs. application responsibility**: Application team sees deployment failure, but root cause is platform issue

**Detection strategy**:
- **Layer 1 (surface symptom)**: Pods stuck in Pending → alert immediately
- **Layer 2 (scheduling constraint)**: TSC blocking scheduling → provide context
- **Layer 3 (autoscaling failure)**: Nodes not joining → escalate to platform team
- Automation should surface all three layers simultaneously

**Remediation safety**:
- TSC softening (ScheduleAnyway) is safe but has trade-offs:
  - ✅ Unblocks deployment
  - ⚠️ Creates uneven zone distribution (potential availability impact)
- Automation could suggest this, but human should approve
- True fix requires platform team to resolve node join issue

### Platform Requirements

**Detection capabilities needed**:
1. **K8s deployment health monitoring**: Track rollout progress, detect stalls
2. **Pod scheduling failure analysis**: Parse pod events, extract reason (TSC, resources, etc.)
3. **Autoscaler integration**: Monitor Karpenter/cluster autoscaler success/failure
4. **Node lifecycle monitoring**: Track node provisioning → Ready state transition
5. **Cross-layer correlation**: Connect pod Pending → node provisioning → kubelet failures

**Diagnosis capabilities needed**:
1. **Multi-layer failure visualization**: Show full stack:
   - Deployment status (rollout progress)
   - Pod status (Pending + reason)
   - Node status (provisioning + join failures)
   - Autoscaler status (Karpenter errors)
2. **TSC constraint analyzer**: Given TSC policy + cluster topology, determine if constraint can be satisfied
3. **Historical correlation**: Has this deployment failed before? Recent platform changes?
4. **Runbook integration**: Link to "node join failure troubleshooting" runbook

**Remediation capabilities needed** (future phase):
1. **TSC softening recommendation**: Suggest safe configuration changes
2. **Platform team escalation**: Auto-create ticket with platform team with full context
3. **Rollback capability**: If deployment stalls >X minutes, suggest rollback option
4. **Manual intervention guidance**: Provide step-by-step commands for operators

**Integration requirements**:
- Access to K8s API: deployments, pods, nodes, events
- Access to Karpenter controller logs/metrics
- Access to node kubelet logs (may require platform team integration)
- Understanding of cluster topology (zones, node labels)
- Historical deployment data (success/failure rates)

**Challenges**:
- Platform-level failures (node join) may be outside application team's observability
- Requires coordination between application monitoring and platform monitoring
- May need platform team to expose additional metrics/logs

### Key Takeaways

1. **Timeline data is critical**: Without timestamps, cannot measure TTD/TTR or identify bottlenecks. This RCA's lack of timeline shows documentation quality varies - standardized RCA templates needed.

2. **Multi-layer failures are hard to diagnose**: Surface symptom (pods pending) is far removed from root cause (node join failures). Automation must surface all layers, not just top-level symptoms.

3. **Hidden platform failures**: Node provisioning failures are invisible to application teams without specific monitoring. Platform integration is essential for full observability.

4. **TSC is a common misconfiguration**: Topology spread constraints are powerful but easy to get wrong. Pre-deployment validation could prevent many TSC-related incidents.

5. **Automation can detect and diagnose, but remediation requires platform fixes**: Unlike RCA #3 (which could be auto-remediated with quota increase), this incident's root cause (node join failures) requires platform team intervention. Automation's role: fast detection, clear diagnosis, smart escalation.

6. **Detection is more valuable than remediation for platform issues**: Since remediation requires platform team, getting fast signal to the right team is more valuable than attempting automated fixes. Alert should say: "Deployment stalled due to node join failures - platform team paged."
