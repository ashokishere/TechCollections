## 1. Executive Summary

Deploying software updates in production requires balancing service availability, release speed, resource utilization, and risk management. In a Kubernetes environment, choosing the correct deployment strategy determines how application updates are distributed across pods and how potential failures impact end users. 

This document analyzes four fundamental Kubernetes deployment strategies: **Recreate**, **Rolling Update**, **Canary**, and **Blue-Green**. Each strategy presents distinct operational trade-offs. Rolling Update serves as the Kubernetes default, offering seamless zero-downtime deployments at low resource cost. Recreate sacrifices availability for absolute simplicity. Canary minimizes blast radius by exposing new versions to a tiny fraction of user traffic before full rollout. Blue-Green provides instant traffic switching and bulletproof rollbacks at the cost of doubling infrastructure overhead. Mastery of these patterns and their underlying traffic management mechanisms enables engineering teams to design resilient, automated delivery pipelines tailored to application requirements.

---

## 2. Key Takeaways

* **Kubernetes Default:** The `RollingUpdate` strategy is the native default in Kubernetes, providing zero-downtime updates with low overhead by incrementally replacing old pods with new ones.
* **Control Mechanics:** Parameters like `maxSurge` and `maxUnavailable` give fine-grained control over pod capacity and resource utilization during a Rolling Update.
* **Downtime vs. Simplicity:** The `Recreate` strategy terminates all running pods before spinning up new ones. While it causes service downtime, it eliminates backward-compatibility requirements for databases and stateful backends.
* **Blast Radius Reduction:** `Canary` deployments route a small percentage of live production traffic (e.g., 5–10%) to a new version, relying on metric monitoring to validate stability before wide release.
* **Instant Rollbacks:** `Blue-Green` deployments run two isolated, identical environments simultaneously. Traffic is switched instantly via Service or Ingress selectors, enabling instant rollbacks if issues arise.
* **Prerequisites for Success:** Zero-downtime strategies depend heavily on accurate Kubernetes Liveness and Readiness Probes, alongside backward-compatible application and database schema designs.

---

## 3. Topics Covered

1. **Introduction to Deployment Strategies**  
   An overview of software release management in Kubernetes and why structured deployment strategies are critical for reliability and uptime.
2. **Recreate Deployment Strategy**  
   A analysis of the all-at-once update mechanism, detailing its process, downtime implications, and ideal non-production use cases.
3. **Rolling Update Deployment Strategy**  
   A detailed breakdown of Kubernetes' default strategy, covering incremental pod replacement, zero-downtime mechanics, and `maxSurge`/`maxUnavailable` parameters.
4. **Canary Deployment Strategy**  
   An investigation into risk-mitigation releases, traffic-splitting techniques using Ingress controllers, and real-time observability requirements.
5. **Blue-Green Deployment Strategy**  
   An examination of dual-environment deployments, service selector flipping, zero-downtime releases, and infrastructural cost implications.
6. **Strategy Comparison & Decision Matrix**  
   A comparative assessment evaluating each strategy across downtime, cost, complexity, and rollback speed to guide architectural selection.

---

## 4. Timeline with Timestamps

- **[00:00]** **Introduction to Deployment Strategies and Why They Are Needed** – Contextualizing software release risks and uptime targets in Kubernetes.
- **[00:45]** **Overview of Key Deployment Strategies** – A high-level survey of Recreate, Rolling Update, Canary, and Blue-Green patterns.
- **[01:30]** **Recreate Deployment Strategy** – Deconstructing the drop-and-replace update pattern and its associated downtime.
- **[03:45]** **Rolling Update Deployment Strategy (Kubernetes Default)** – Inspecting continuous availability through incremental pod replacement and surge parameters.
- **[09:30]** **Canary Deployment Strategy** – Implementing traffic splitting, ingress management, and metric verification for risk control.
- **[16:45]** **Blue-Green Deployment Strategy** – Analyzing dual-environment routing, service-selector switching, and state management challenges.
- **[24:00]** **Comparison of Deployment Strategies** – Synthesizing trade-offs between cost, latency, complexity, and safety.
- **[26:30]** **Conclusion and Further Resources** – Summary of best practices and implementation tooling.

---

## 5. Detailed Explanation

### Recreate Deployment Strategy
The **Recreate** strategy is the simplest update mechanism. When an update is triggered, the deployment controller terminates all running pods belonging to version 1 ($V_1$). Only after all $V_1$ pods are completely terminated does the controller create new pods running version 2 ($V_2$).

```
[ All V1 Pods Terminated ] ---> [ Downtime Window ] ---> [ V2 Pods Created & Ready ]
```

* **Lifecycle:** $V_1$ running $\rightarrow$ Scaled down to $0$ replicas $\rightarrow$ System unavailable $\rightarrow$ $V_2$ scaled up to $N$ replicas $\rightarrow$ $V_2$ running.
* **Advantages:** Absolute simplicity. Eliminates the possibility of two different code versions running concurrently, preventing API mismatch issues or complex database schema migration locks.
* **Disadvantages:** Significant application downtime during the transition period (proportional to container pull and startup times).

### Rolling Update Deployment Strategy
The **Rolling Update** strategy incrementally updates pod instances from $V_1$ to $V_2$. It guarantees that a minimum number of pods remain available to handle user requests throughout the deployment process, providing zero-downtime updates.

```
Step 1: [V1] [V1] [V1] [V1]  (100% V1)
Step 2: [V1] [V1] [V1] [V2]  (25% V2, 75% V1)
Step 3: [V1] [V1] [V2] [V2]  (50% V2, 50% V1)
Step 4: [V2] [V2] [V2] [V2]  (100% V2)
```

* **Lifecycle:** A new ReplicaSet ($RS_2$) is created alongside the existing ReplicaSet ($RS_1$). $RS_2$ scales up by a small batch while $RS_1$ scales down proportionally.
* **Pacing Parameters:** Controlled via `maxSurge` (how many extra pods can be created above target replica count) and `maxUnavailable` (how many pods can be deleted below target replica count).
* **Advantages:** Zero downtime, low resource overhead (requires only incremental surge capacity), native Kubernetes support without extra tools.
* **Disadvantages:** Rollout is gradual. $V_1$ and $V_2$ run concurrently, requiring strict API backward compatibility and stateless application designs.

### Canary Deployment Strategy
The **Canary** deployment strategy routes a small percentage of live production traffic to a new software release ($V_2$) while keeping the majority on the stable release ($V_1$). The canary instance is monitored for errors, latency spikes, and log anomalies.

```
                      /---> [ 90% Traffic ] ---> [ Deployment V1 (Stable) ]
[ Ingress Router ] --|
                      \---> [ 10% Traffic ] ---> [ Deployment V2 (Canary) ]
```

* **Lifecycle:** $V_2$ is deployed with a small replica count. An Ingress controller or Service Mesh (e.g., NGINX Ingress, Istio) splits incoming traffic by ratio (e.g., 90/10). If canary telemetry is healthy, traffic is incrementally shifted (20%, 50%, 100%) until $V_1$ is decommissioned.
* **Advantages:** Minimizes blast radius; real-world production verification; rapid rollbacks by shifting traffic back to $V_1$.
* **Disadvantages:** High implementation complexity; requires traffic management CRDs/controllers and sophisticated observability pipelines.

### Blue-Green Deployment Strategy
The **Blue-Green** strategy runs two full, independent production environments side by side. Environment **Blue** hosts the current live release, while environment **Green** hosts the new version.

```
                           [ Ingress / Service Selector ]
                                  |         |
                  (Live Traffic) /          \ (Idle / Testing)
                                v            v
                       [ Blue: V1 ]       [ Green: V2 ]
```

* **Lifecycle:** Green ($V_2$) is deployed in complete isolation. Full integration, regression, and load testing are executed against Green without impacting live users. Once verified, the Service selector or Ingress rule is modified to point instantly to Green.
* **Advantages:** Instant switchover (zero downtime); instantaneous rollback (flip selector back to Blue); complete isolation during testing.
* **Disadvantages:** High cost (requires double capacity); complex stateful data and database synchronization requirements.

---

## 6. Beginner Explanation (ELI5)

Imagine you run a popular 24-hour restaurant that needs to update its menu and kitchen equipment:

1. **Recreate (The Renovator):**  
   You close the restaurant completely, lock the doors, throw away old stoves, install new ones, train the staff, and reopen 12 hours later. Customers are turned away during the work, but there is zero confusion inside the kitchen.

2. **Rolling Update (The Shift Switcher):**  
   You replace one cook and one stove at a time while keeping the kitchen open. As soon as the new cook gets up to speed, you replace the next one. The restaurant stays open, but for a short period, half the kitchen is cooking the old menu and half is cooking the new menu.

3. **Canary (The Taste Tester):**  
   You keep the kitchen running normally on the old menu, but you install one small new stove in the corner. You invite 5% of your walk-in customers to try meals from the new menu. If they love it and don't get sick, you gradually upgrade the rest of the kitchen.

4. **Blue-Green (The Twin Restaurants):**  
   You build an exact replica of your restaurant right next door ("Green"). You test everything inside Green until it is flawless. On opening night, you flip a giant lighted sign pointing customers from the old building ("Blue") to the new building ("Green"). If a pipe bursts in the new building, you immediately flip the sign back to the old building.

---

## 7. Technical Deep Dive

### Kubernetes Deployment Controller Architecture
When a Deployment manifest is updated, the Kubernetes **Deployment Controller** does not directly edit Pods. Instead, it manages **ReplicaSets**. 

In a **Rolling Update**:
1. The Deployment Controller creates a new ReplicaSet ($RS_{\text{new}}$).
2. It calculates maximum allowable pods based on parameters:
   $$\text{Max Allowed Pods} = \text{replicas} + \text{maxSurge}$$
   $$\text{Min Available Pods} = \text{replicas} - \text{maxUnavailable}$$
3. $RS_{\text{new}}$ scales up, triggering the Kubernetes scheduler to place pods.
4. Pods transition through container runtime creation, entrypoint execution, Liveness probe initialization, and Readiness probe evaluation.
5. Once a pod in $RS_{\text{new}}$ returns HTTP 200/TCP success on its **Readiness Probe**, it is registered into the endpoints object of matching Services.
6. The old ReplicaSet ($RS_{\text{old}}$) scales down by one replica.
7. This loop repeats until $RS_{\text{old}}$ reaches $0$ replicas and $RS_{\text{new}}$ reaches the target `replicas` count.

```
Deployment
 ├── ReplicaSet V1 (Scale Down) --> [ Pod V1 ] [ Pod V1 ]
 └── ReplicaSet V2 (Scale Up)   --> [ Pod V2 (Passing Readiness) ]
```

### Advanced Traffic Splitting Mechanics
Standard Kubernetes Services load-balance traffic uniformly across all pods matching a label selector using IPVS or `iptables` rules. Achieving fractional traffic splitting (e.g., 90/10 Canary) with native Services requires deploying pods in exact numerical ratios (e.g., 9 pods $V_1$, 1 pod $V_2$).

To achieve precise, byte-level or request-level Canary releases independent of pod counts, ingress controllers or Service Meshes operate at Layer 7:
* **Ingress-NGINX:** Uses custom annotations (`nginx.ingress.kubernetes.io/canary-weight`) to dynamically adjust NGINX upstream weight configurations without scaling pod counts.
* **Service Mesh (Istio / Envoy):** Intercepts traffic via sidecar proxies. Istio `VirtualService` rules specify percentage routes at the application layer:

```
spec:
  http:
  - route:
    - destination:
        host: my-service
        subset: v1
      weight: 90
    - destination:
        host: my-service
        subset: v2
      weight: 10
```

### Database Schemas in Non-Recreate Deployments
Whenever $V_1$ and $V_2$ coexist (Rolling Update, Canary, Blue-Green), the database schema must support **both** versions simultaneously. This requires the **Expand and Contract Pattern**:
1. **Expand:** Add new columns/tables as optional or nullable. Deploy $V_2$ code that writes to both old and new schema structures.
2. **Migrate:** Backfill old data into the new schema structure.
3. **Contract:** Once $V_1$ is decommissioned across all nodes, deploy a clean-up update removing legacy code paths, then drop old database columns/tables.

---

## 8. Important Definitions

* **Deployment:** A declarative Kubernetes API object that manages the state, scaling, and update strategy of a set of identical Pods.
* **ReplicaSet:** A lower-level controller that guarantees a specified number of identical running Pods exist at any given time.
* **Readiness Probe:** A health check used by Kubernetes to determine if a container is ready to accept user traffic.
* **Liveness Probe:** A health check used by Kubernetes to determine if a container needs to be restarted.
* **maxSurge:** An absolute number or percentage defining how many additional pods can be created above the desired replica count during an update.
* **maxUnavailable:** An absolute number or percentage defining how many pods can be rendered unavailable below the desired replica count during an update.
* **Canary Deployment:** A release pattern where software is rolled out to a tiny subset of users before full infrastructure distribution.
* **Blue-Green Deployment:** A release pattern deploying two identical environments, using traffic-switching to instantly shift all traffic between them.
* **Ingress Controller:** An application (e.g., NGINX, Traefik) that manages external HTTP/HTTPS access to services within a Kubernetes cluster.

---

## 9. Code Snippets & Configuration Examples

### 1. Recreate Strategy Manifest (`recreate-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-recreate
  labels:
    app: my-app
    strategy: recreate
spec:
  replicas: 4
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        version: "2.0.0"
    spec:
      containers:
      - name: web
        image: nginx:1.25.3
        ports:
        - containerPort: 80
```

### 2. Rolling Update Strategy Manifest (`rolling-deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-rolling
  labels:
    app: my-app
spec:
  replicas: 8
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%       # Temporarily scales up to 10 pods max
      maxUnavailable: 0%  # Ensures 100% (8 pods) capacity is maintained throughout
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        version: "2.1.0"
    spec:
      containers:
      - name: web
        image: myregistry.internal/api-server:v2.1.0
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /live
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
```

### 3. Canary Ingress Routing Configuration (NGINX Ingress)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-canary-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10" # Directs 10% of traffic to Canary
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service-canary
            port:
              number: 80
```

### 4. Blue-Green Service Switching Manifest (`service-blue-green.yaml`)
```yaml
# Step 1: Deploy app-blue and app-green deployments separately.
# Step 2: Update selector below to switch environment instantly.
apiVersion: v1
kind: Service
metadata:
  name: web-production-service
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: my-app
    color: green  # Change from 'blue' to 'green' to flip entire live traffic stream
```

---

## 10. Best Practices

1. **Always Define Readiness Probes:** Never run zero-downtime updates without explicit Readiness Probes. Without them, Kubernetes routes traffic to newly spun-up pods before the application entrypoint has finished booting.
2. **Enforce Immutable Tags:** Avoid using `:latest` image tags. Use strict semantic versioning or git commit SHAs (e.g., `image: myapp:v1.4.2` or `myapp:a1b2c3d`) to ensure reliable rollbacks.
3. **Configure Conservative `maxSurge` / `maxUnavailable` Parameters:** For strict zero-downtime under high traffic, set `maxUnavailable: 0` and `maxSurge: 25%` to guarantee existing operational capacity is never throttled during releases.
4. **Automate Metric Verification:** Pair Canary deployments with progressive delivery tools like **Argo Rollouts** or **Flagger** linked directly to Prometheus metrics (HTTP 5xx rate, latency p99) for automated rollbacks.
5. **Decouple Database Schema Migrations:** Implement non-breaking database schema changes (Expand/Contract pattern) prior to initiating application code deployments.

---

## 11. Common Mistakes

* **Missing Resource Quota Margins:** Setting `maxSurge: 50%` or using Blue-Green deployments on a cluster running near capacity ($>80\%$ memory/CPU utilization). Pods will stall in a `Pending` state due to insufficient cluster resources.
* **Mixing Up Liveness and Readiness Probes:** Configuring a Liveness probe that fails fast during startup, causing Kubernetes to endlessly kill and restart pods during a Rolling Update loop.
* **Stateful Applications with Rolling Updates:** Executing rolling updates on stateful services that write to local file systems or run single-instance background schedulers, causing split-brain scenarios or data corruption.
* **Forgetting Client-Side Caching / Connection Pooling:** Long-lived persistent TCP connections or browser caching can prevent users from picking up new versions or HTTP traffic routes immediately after a Blue-Green flip.

---

## 12. Interview Questions

### Q1: Explain the functional difference between `maxSurge` and `maxUnavailable` in a Kubernetes RollingUpdate strategy.
**Answer:** `maxSurge` specifies the maximum number of Pods that can be created **above** the desired replica count specified in the Deployment. It can be an absolute integer or a percentage. `maxUnavailable` specifies the maximum number of Pods that can be **unavailable** below the target replica count during the update process. Setting `maxUnavailable: 0` guarantees full baseline servicing capacity throughout the release, but requires extra cluster resource headroom for the surging pods.

### Q2: How does a Kubernetes Service know when to stop sending traffic to a V1 pod and start sending to a V2 pod during a Rolling Update?
**Answer:** Kubernetes relies on the **Readiness Probe**. During a rolling update, as V2 pods boot up, Kubernetes probes their readiness endpoint. Traffic is routed to a V2 pod only after its readiness probe returns a successful status. Concurrently, when a V1 pod is scheduled for termination, Kubernetes removes its IP address from the Service's `Endpoints` (or `EndpointSlice`) object, sends a `SIGTERM` signal to the container, and waits for `terminationGracePeriodSeconds` before issuing a `SIGKILL`.

### Q3: Why is a Blue-Green deployment more expensive than a Rolling Update, and what problem does it solve best?
**Answer:** A Blue-Green deployment requires running two entirely complete production environments simultaneously, effectively doubling infrastructure resource costs during deployment. It solves two key problems: it enables instant zero-downtime cutover without intermediate states where V1 and V2 coexist, and it allows comprehensive production-identical testing on the Green environment prior to exposing live traffic.

### Q4: If a Canary deployment exhibits a high 5xx error rate, how is the failure contained?
**Answer:** Because traffic splitting limits the canary instance to a minor fraction of overall user traffic (e.g., 5%), the blast radius is restricted to that small percentage. The issue is contained by immediately adjusting the ingress traffic routing rules back to 100% for the stable V1 deployment and scaling down the canary deployment, resolving the incident before it impacts the broader user base.

---

## 13. Certification Questions

### Question 1 (CKAD)
A developer needs to ensure that during a deployment update, the cluster never scales below the target replica count of 4 pods, but allows up to 2 additional temporary pods during the rollout. Which configuration fragment is correct?

- A) 
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 0
    maxUnavailable: 2
```
- B) 
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 2
    maxUnavailable: 0
```
- C) 
```yaml
strategy:
  type: Recreate
  recreate:
    maxSurge: 2
```
- D) 
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 50%
    maxUnavailable: 50%
```

**Correct Answer:** **B**  
**Explanation:** `maxUnavailable: 0` guarantees the number of available pods never drops below the 4-pod baseline. `maxSurge: 2` permits up to 2 extra pods (6 total) during the rollout process.

---

### Question 2 (CKA)
When applying a Blue-Green strategy in native Kubernetes using basic objects, which API object selector is typically modified to execute the environment traffic cutover?

- A) The Ingress host matching rule
- B) The ConfigMap data property
- C) The ClusterIP Service's `spec.selector`
- D) The Deployment's `spec.template.spec.nodeSelector`

**Correct Answer:** **C**  
**Explanation:** A standard native Kubernetes Blue-Green deployment updates the `spec.selector` key-value pairs of the exposing Service (e.g., changing `color: blue` to `color: green`) to immediately point the cluster service load balancer to the new set of pods.

---

## 14. Real-World Examples

1. **E-Commerce Checkout Service (Rolling Update):**  
   An online retailer updates its payment processing microservice during high-volume periods using a `RollingUpdate` with `maxSurge: 25%` and `maxUnavailable: 0%`. This guarantees zero dropped carts while maintaining continuous backend processing capability.

2. **Core Banking Transaction API (Blue-Green):**  
   A financial institution requires strict end-to-end integration and security testing on new API builds before exposing them to users. They provision a secondary Green environment, perform synthetic security tests, and flip the DNS/Service routing instantly at midnight.

3. **Global Streaming Platform (Canary via Service Mesh):**  
   A video streaming engine rolls out a new encoding microservice. Using Istio and Flagger, they route 2% of user requests to the canary pod. Automated Prometheus telemetry tracks playback buffering rates; if buffering increases by >0.1%, the release automatically rolls back without human intervention.

---

## 15. Analogies

* **Recreate Strategy:** **Replacing a Bridge.** Closing a bridge completely to demolish it and build a new bridge in its place. Traffic stops entirely until the new bridge is fully constructed and opened.
* **Rolling Update Strategy:** **Escalator Step Replacement.** Fixing an escalator step-by-step while it keeps moving. Individual steps are pulled out and swapped one at a time while passengers continue riding on the remaining steps.
* **Blue-Green Strategy:** **Airport Runway Switch.** A major airport builds a brand new parallel runway (Green) while airplanes continue landing on the existing runway (Blue). Once Green is checked, air traffic control instructs incoming flights to land on the new runway.

---

## 16. Frequently Asked Questions

### Q: Which deployment strategy is selected by default in Kubernetes?
**A:** `RollingUpdate` is the native default strategy when no strategy type is explicitly defined in a Deployment manifest.

### Q: Can I run a Canary deployment natively with standard Kubernetes Deployment objects?
**A:** Partially. You can run two separate Deployments ($V_1$ and $V_2$) sharing a single Service label selector. However, traffic ratio control is coarse and proportional to pod count ratios (e.g., 9 pods $V_1$ vs 1 pod $V_2$). Precise percentage-based traffic splitting requires an Ingress Controller (like NGINX) or Service Mesh (like Istio).

### Q: What happens if a rollout fails midway during a Rolling Update?
**A:** Kubernetes pauses the rollout if newly created pods fail their Readiness or Liveness probes. Old pods remain active to handle traffic. You can inspect the status using `kubectl rollout status` and revert immediately using `kubectl rollout undo deployment/<deployment-name>`.

### Q: Does a Blue-Green deployment require double the cluster hardware?
**A:** Yes. Because both Blue and Green environments run concurrently at full scale during verification and cutover, your cluster must have sufficient compute resources (CPU/Memory) to host both deployment stacks simultaneously.

---

## 17. Related Technologies

* **Argo Rollouts:** A Kubernetes controller and set of CRDs that bring advanced deployment capabilities such as Canary, Blue-Green, experiments, and progressive delivery analysis to Kubernetes.
* **Flagger:** A progressive delivery tool that automates the release process for applications running on Kubernetes using service mesh (Istio, Linkerd) or ingress controllers.
* **NGINX Ingress Controller:** An ingress implementation that supports advanced traffic-splitting annotations for Canary releases.
* **Istio / Linkerd:** Service Mesh architectures providing L7 traffic management, fine-grained routing, mutual TLS, and telemetry collection required for advanced Canary deployments.

---

## 18. Important Quotes

> *"Choosing a deployment strategy is a continuous exercise in balancing risk, availability, cost, and structural complexity."*

> *"Without correctly configured Readiness Probes, zero-downtime deployment strategies are only zero-downtime on paper."*

> *"If your database schema changes are not backward-compatible, no Kubernetes strategy in the world can save you from downtime or data corruption."*

---

## 19. Glossary

| Term | Definition |
| :--- | :--- |
| **Blue-Green Deployment** | Strategy utilizing two identical isolated environments, switching traffic instantly via routing controls. |
| **Canary Deployment** | Strategy rolling out changes to a small subset of users to test stability before widespread deployment. |
| **maxSurge** | Parameter controlling how many extra pods can be provisioned above target replica count during a rollout. |
| **maxUnavailable** | Parameter controlling how many pods can be rendered offline below target replica count during a rollout. |
| **Readiness Probe** | Health check indicating whether a container is ready to accept network traffic. |
| **ReplicaSet** | Kubernetes controller ensuring a specified number of pod replicas are running at any given time. |
| **Recreate Strategy** | Strategy terminating all existing pods before starting new version instances, causing downtime. |
| **RollingUpdate Strategy** | Strategy gradually replacing old pods with new ones without taking the application offline. |

---

## 20. One-Page Cheat Sheet

### Strategy Comparison Matrix

| Strategy | Downtime | Resource Cost | Rollback Speed | Complexity | Blast Radius |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Recreate** | High | Low ($1\times$) | Slow | Very Low | Entire User Base |
| **Rolling Update** | Zero | Low-Medium ($1\times + \text{surge}$) | Medium | Low | Partial |
| **Canary** | Zero | Medium ($1\times + \text{canary}$) | Fast | High | Very Small (Target Group) |
| **Blue-Green** | Zero | High ($2\times$) | Instant | Medium | Small (Pre-tested) |

### Key `kubectl` Management Commands

```bash
# Check status of an ongoing rollout
kubectl rollout status deployment/my-app

# View deployment revision history
kubectl rollout history deployment/my-app

# Roll back to previous revision immediately
kubectl rollout undo deployment/my-app

# Roll back to a specific historical revision
kubectl rollout undo deployment/my-app --to-revision=2

# Update image triggering a rolling update
kubectl set image deployment/my-app web=myregistry/nginx:v2.0 --record
```

---

## 21. Flash Cards

- **Card 1 | Deployment Basics**
  - **Q:** What is the default deployment strategy in Kubernetes?
  - **A:** `RollingUpdate`.

- **Card 2 | Parameters**
  - **Q:** What happens if `maxUnavailable` is set to `0%` during a Rolling Update?
  - **A:** Kubernetes will not terminate any old pods until new pods are created, running, and passing their Readiness Probes, guaranteeing 100% capacity throughout the rollout.

- **Card 3 | Release Strategies**
  - **Q:** Which strategy requires running two complete identical production environments simultaneously?
  - **A:** Blue-Green Deployment Strategy.

- **Card 4 | Health Checks**
  - **Q:** Which probe prevents Kubernetes from sending traffic to a newly created pod during an update rollout?
  - **A:** The Readiness Probe.

- **Card 5 | Traffic Splitting**
  - **Q:** What primary advantage does a Canary release offer over a standard Rolling Update?
  - **A:** It minimizes the blast radius by limiting exposure of the new release to a tiny fraction of live user traffic while operational metrics are validated.

---

## 22. Quiz

### Q1: What is the primary disadvantage of the Recreate deployment strategy?
- A) High cost due to duplicate infrastructure.
- B) High risk of traffic splitting errors.
- C) Application downtime while old pods are terminated and new ones start up.
- D) Need for complex service mesh integration.  
**Correct Answer:** **C**  
**Explanation:** Recreate terminates all existing pods before creating new ones, leading to inevitable downtime between the termination and initialization steps.

---

### Q2: By default, what are the standard values for `maxSurge` and `maxUnavailable` in Kubernetes Deployments?
- A) `maxSurge: 50%`, `maxUnavailable: 50%`
- B) `maxSurge: 25%`, `maxUnavailable: 25%`
- C) `maxSurge: 0`, `maxUnavailable: 100%`
- D) `maxSurge: 1`, `maxUnavailable: 0`  
**Correct Answer:** **B**  
**Explanation:** Kubernetes defaults both `maxSurge` and `maxUnavailable` to `25%` of the target replica count.

---

### Q3: In a Rolling Update, when is an old pod safely scaled down?
- A) Immediately after the new pod container is created.
- B) As soon as the new pod reaches `Running` status, regardless of health checks.
- C) After the new pod passes its Readiness Probe.
- D) After `terminationGracePeriodSeconds` expires.  
**Correct Answer:** **C**  
**Explanation:** Kubernetes waits for new pods to pass their Readiness Probes before considering them active and proceeding to terminate old pods.

---

### Q4: Which deployment strategy provides the FASTEST rollback mechanism?
- A) Recreate
- B) Rolling Update
- C) Blue-Green
- D) Re-apply Manifest  
**Correct Answer:** **C**  
**Explanation:** Blue-Green rollbacks are instant because the old environment (Blue) remains fully running in standby mode; reverting requires only updating the Service selector or DNS/Ingress mapping back to Blue.

---

### Q5: What database design pattern is required to support concurrent $V_1$ and $V_2$ application instances during a Rolling Update?
- A) Sharding
- B) Expand and Contract Pattern
- C) ACID Isolation Level 4
- D) Read-Only Replica Mirroring  
**Correct Answer:** **B**  
**Explanation:** The Expand and Contract pattern allows database schemas to support both old ($V_1$) and new ($V_2$) application versions concurrently without breaking database queries.

---

### Q6: If a cluster lacks sufficient compute resource quota, what issue occurs during a Rolling Update with `maxSurge: 50%`?
- A) Old pods are immediately deleted.
- B) New pods get stuck in a `Pending` state.
- C) The deployment automatically converts to Blue-Green.
- D) Kubernetes forces node autoscaling to bypass quotas.  
**Correct Answer:** **B**  
**Explanation:** `maxSurge` requests extra pod scheduling. If the cluster lacks capacity, the scheduler cannot place the surged pods, leaving them in a `Pending` state.

---

### Q7: Which component is essential for fine-grained, percentage-based Canary traffic distribution in Kubernetes?
- A) kube-scheduler
- B) Ingress Controller / Service Mesh
- C) CoreDNS
- D) kube-proxy IPVS Mode  
**Correct Answer:** **B**  
**Explanation:** Layer 7 Ingress Controllers (like NGINX) or Service Meshes (like Istio) are required to split traffic by exact percentages independent of pod ratios.

---

### Q8: Why would a team deliberately choose a Recreate deployment strategy over RollingUpdate?
- A) To achieve zero downtime.
- B) To run deployments faster on large clusters.
- C) To avoid running two different code versions simultaneously when handling non-backward-compatible changes.
- D) To eliminate the need for container images.  
**Correct Answer:** **C**  
**Explanation:** Recreate ensures that $V_1$ and $V_2$ never run at the same time, avoiding complex data or API compatibility issues.

---

### Q9: What happens if a Deployment specifies `maxUnavailable: 0` and `maxSurge: 0`?
- A) Kubernetes sets default values automatically.
- B) Kubernetes accepts the configuration and performs an instant update.
- C) The API server rejects the Deployment validation error.
- D) The Deployment performs a Canary rollout.  
**Correct Answer:** **C**  
**Explanation:** Kubernetes validation rules prevent both `maxSurge` and `maxUnavailable` from being set to `0` simultaneously, as no pod changes could occur.

---

### Q10: What command checks the real-time progress of an active Kubernetes deployment update?
- A) `kubectl get pods --watch`
- B) `kubectl rollout status deployment/<name>`
- C) `kubectl describe service/<name>`
- D) `kubectl cluster-info update`  
**Correct Answer:** **B**  
**Explanation:** `kubectl rollout status` monitors deployment rollout progress and reports success or failure states.

---

## 23. Action Items

- [ ] **Audit Health Checks:** Inspect all production Deployment manifests to ensure both `readinessProbe` and `livenessProbe` definitions exist and are properly tuned.
- [ ] **Tune Surge Parameters:** Review baseline capacity needs and update `maxSurge` and `maxUnavailable` values based on cluster headroom and availability SLAs.
- [ ] **Implement Immutable Tagging:** Remove `:latest` container tags across CI/CD delivery pipelines and replace them with unique git commit SHAs or SemVer strings.
- [ ] **Test Emergency Rollback Procedures:** Practice executing `kubectl rollout undo deployment/<name>` in a non-production staging environment to verify rollback behavior.
- [ ] **Evaluate Progressive Delivery Tooling:** Explore advanced tools like **Argo Rollouts** or **Flagger** if canary deployments with automated metrics validation are required.

---

## 24. Recommended Further Reading

* [Kubernetes Official Documentation – Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Kubernetes Official Documentation – Service and Endpoint Management](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Argo Rollouts Architecture & Architecture Patterns](https://argoproj.github.io/argo-rollouts/)
* [Flagger Canary Deployments Documentation](https://flagger.app/)
* *Kubernetes Patterns: Reusable Elements for Designing Cloud-Native Applications* by Bilgin Ibryam & Roland Huß (O'Reilly Media)