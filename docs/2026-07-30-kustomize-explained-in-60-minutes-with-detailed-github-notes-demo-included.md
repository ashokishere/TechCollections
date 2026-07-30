## 1. Executive Summary

In this comprehensive tutorial, Abhishek Veeramalla breaks down **Kustomize**, a native Kubernetes declarative configuration management tool. Managing Kubernetes YAML manifests across diverse deployment environments—such as Development, Staging, and Production—frequently leads to code duplication, configuration drift, and maintenance overhead. Kustomize resolves these challenges by introducing a template-free, patch-based configuration strategy rooted in a **Base and Overlays** model.

Rather than relying on complex templating engines like Helm, Kustomize works directly with standard, plain-text Kubernetes YAML manifests. Base directories hold invariant resource definitions (e.g., deployments, services), while Overlay directories apply target-specific patches, label injections, and resource generators (e.g., `configMapGenerator`). Built directly into `kubectl` via the `-k` flag, Kustomize enables pure declarative configuration management, preserving a single source of truth across GitOps pipelines.

---

## 2. Key Takeaways

* **Template-Free Customization**: Customizes raw, valid Kubernetes YAML manifests without introducing complex templating engines or non-standard syntax.
* **Base and Overlays Pattern**: Promotes reuse by establishing a single base manifest directory layered beneath environment-specific overlay patches (e.g., `dev`, `staging`, `prod`).
* **Native `kubectl` Integration**: Directly integrated into the Kubernetes CLI through commands like `kubectl apply -k <directory>`, eliminating external dependencies.
* **Granular Patching**: Supports both Strategic Merge Patches (declarative YAML merging) and JSON Patches (RFC 6902 precise mutations).
* **Automatic ConfigMap & Secret Hash Suffixing**: Generates unique append hashes for `ConfigMap` and `Secret` resources, guaranteeing pod restarts whenever configuration data changes.
* **Built-in Transformers**: Effortlessly applies cross-cutting modifications, such as `namePrefix`, `namespace`, `commonLabels`, and `commonAnnotations`.

---

## 3. Topics Covered

* **Problem Statement: Configuration Drift & Duplication**: Explores why maintaining separate raw YAML copies across environments causes divergence and operational overhead.
* **Introduction to Kustomize**: Defines Kustomize as a native, declarative, template-free configuration tool.
* **Kustomize vs. Helm**: Compares Kustomize’s patch-based approach against Helm’s chart-and-value template engine.
* **Directory Structure (Base & Overlays)**: Demonstrates how to organize code into reusable base components and environment-specific overlays.
* **The `kustomization.yaml` Specification**: Details the core fields, API versions, resource declarations, and schema layout of configuration files.
* **Patching Mechanics**: Explains Strategic Merge Patching versus explicit JSON Patch (RFC 6902) execution.
* **Generators (`configMapGenerator` & `secretGenerator`)**: Covers automated creation of dynamic Kubernetes configuration objects from local files or literals.
* **CLI Execution & Operations**: Practical usage of `kustomize build` and direct deployment via `kubectl apply -k`.
* **Advanced Features & Best Practices**: Explores built-in transformers, GitOps integration patterns, and organizational standards.

---

## 4. Timeline with Timestamps

- **[00:00]** **Introduction to Kustomize and the Problem it Solves**: Challenges of managing multi-environment Kubernetes configs and configuration drift.
- **[00:05]** **Why Kustomize? Use Cases and Benefits**: Template-free approach, plain YAML debugging, and multi-namespace deployments.
- **[00:10]** **Kustomize vs. Helm: A Brief Comparison**: Key differences between template substitution (Helm) and structure patching (Kustomize).
- **[00:15]** **Getting Started: Basic Kustomize Structure**: Defining the directory hierarchy: organizing `base/` and `overlays/`.
- **[00:20]** **The `kustomization.yaml` File**: Structuring resources, target references, and `apiVersion: kustomize.config.k8s.io/v1beta1`.
- **[00:25]** **Implementing Overlays**: Layering dev/prod environments on top of shared base resources.
- **[00:35]** **Patches: Strategic Merge and JSON Patch**: Deep dive into overriding specs, replicas, images, and environment variables.
- **[00:45]** **Generators: ConfigMap and Secret Generation**: Creating dynamic `ConfigMaps` and `Secrets` from `.env` files and literals.
- **[00:50]** **Running Kustomize and Applying Configurations**: Hands-on execution with `kustomize build` and `kubectl apply -k`.
- **[00:55]** **Advanced Kustomize Features and Best Practices**: Utilizing transformers (`namePrefix`, `commonLabels`) and maintaining clean repo structures.
- **[01:00]** **Conclusion and Further Learning**: Overview of takeaways, repository walkthrough, and next steps for practice.

---

## 5. Detailed Explanation

### Problem Statement: Configuration Drift & Duplication
Deploying microservices across multiple environments (Development, Staging, Production) requires minor configuration changes per environment—such as replica counts, CPU/memory limits, image tags, and environment variables. Copy-pasting raw Kubernetes manifest files for each environment creates massive duplication. If a core service specification changes, operators must manually update every redundant YAML file, introducing human error and configuration drift.

### Kustomize vs. Helm
* **Helm**: Operates as a package manager that uses Go templating (`{{ .Values.image.repository }}`) to inject values into parameterizes manifests at render time. Helm requires managing chart dependencies, template syntax, and Release state storage within the cluster.
* **Kustomize**: Operates as a declarative configuration engine that leaves base manifests as standard, valid Kubernetes YAML. Instead of templating text, Kustomize parses YAML structures into abstract syntax trees (AST) and applies specific patches over base manifests, generating fully populated YAML outputs.

```
       HELM (Template Substitution)
       Values.yaml + Template.yaml ---> [Engine] ---> Rendered Manifest

       KUSTOMIZE (Overlay Patching)
       Base YAML + Overlay Patch YAML ---> [Engine] ---> Rendered Manifest
```

### The Base and Overlays Concept
* **Base Directory**: Contains invariant, baseline resources (`deployment.yaml`, `service.yaml`, `ingress.yaml`) that define the fundamental structure of an application, along with a primary `kustomization.yaml`.
* **Overlays Directory**: Contains environment-specific subdirectories (e.g., `overlays/dev`, `overlays/prod`). Each overlay contains its own `kustomization.yaml` pointing back to the `base` path, combined with patch files defining modifications (e.g., increasing replica count from 1 to 5).

### Strategic Merge Patch vs. JSON Patch
* **Strategic Merge Patch**: Uses standard Kubernetes YAML syntax to declare target changes. Kustomize merges the patch into the base manifest using the resource's `apiVersion`, `kind`, and `metadata.name` as matching keys.
* **JSON Patch (RFC 6902)**: Enables explicit, atomic operations (`add`, `remove`, `replace`) on specified JSON paths within the targeted resource manifest, providing exact control over modification targets.

### Dynamic Resource Generators
Manually managing `ConfigMap` and `Secret` manifests can be problematic: updating data within a `ConfigMap` does not automatically trigger a rolling update of dependent Pods. Kustomize provides `configMapGenerator` and `secretGenerator`. When executed, Kustomize appends a unique content hash to the resource name (e.g., `my-config-h8f4t9g7k2`). Any change in key-value content alters the hash, changing the deployment spec's referenced `ConfigMap` name and triggering an automatic rolling pod update.

---

## 6. Beginner Explanation (ELI5)

Imagine you are coloring in a house blueprint:

* **The Base**: This is the master drawing of the house. It shows where the walls, doors, windows, and roof go. You never want to write directly on this master drawing because you might want to reuse it for different houses.
* **The Overlay**: Instead of drawing on the master plan, you lay a clear plastic sheet over it. On this plastic sheet, you draw a blue roof for a beach house or a red roof for a mountain cabin.
* **Kustomize**: Kustomize acts like a smart copier. It stacks your clear plastic sheet (Overlay) on top of your master blueprint (Base), presses print, and gives you a single finished paper showing the final customized house.

You don't need complicated formulas or special code tools—you just layer your desired changes directly over the original template!

---

## 7. Technical Deep Dive

### Architecture & Manifest Pipeline
When Kustomize processes a target directory (e.g., via `kustomize build overlays/prod`), it executes a deterministic, multi-stage processing pipeline:

```
+-------------------------------------------------------------------+
| 1. LOAD & PARSE BASE RESOURCES                                    |
|    - Read base/kustomization.yaml                                 |
|    - Parse raw YAML files into Kubernetes Resource Model (KRM)    |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 2. RUN RESOURCE GENERATORS                                        |
|    - Process configMapGenerator & secretGenerator                 |
|    - Generate content hashes & create dynamic resources           |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 3. APPLY TRANSFORMERS                                             |
|    - Apply namePrefix, nameSuffix, namespace                      |
|    - Inject commonLabels and commonAnnotations                    |
|    - Process custom transformers (e.g., image tag mutation)       |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 4. EXECUTE PATCHES                                                |
|    - Apply Strategic Merge Patches                                |
|    - Execute JSON Patches (RFC 6902)                              |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
| 5. RESOLVE REFERENCES & EMIT MANIFEST                             |
|    - Update name references (e.g., deployment -> configMap hash)  |
|    - Serialize final KRM object graph into standard YAML stream   |
+-------------------------------------------------------------------+
```

### Strategic Merge Patch Mechanics
Strategic Merge Patch leverages the Kubernetes API schema's structural metadata (`patchStrategy` and `patchMergeKey`). For example, when updating container arrays in a deployment:

```yaml
# Strategic merge uses 'name' as patchMergeKey for container lists
containers:
- name: web-app
  image: nginx:1.25-alpine
```

Kustomize matches the container named `web-app` in the base manifest and updates its `image` field, preserving other unmentioned container parameters (such as `ports` or `volumeMounts`).

### JSON Patching (RFC 6902)
JSON Patch operations bypass Kubernetes merge keys, directly modifying the document tree target:

```yaml
patches:
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: my-app
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: custom-registry.io/app:v2.0.0
```

---

## 8. Important Definitions

* **Base**: A directory containing standard Kubernetes manifests along with a `kustomization.yaml` file that serves as the root configuration.
* **Overlay**: A directory containing environment-specific patches and settings layered over a base configuration.
* **`kustomization.yaml`**: The primary configuration file consumed by Kustomize to process, modify, and build Kubernetes manifests.
* **Strategic Merge Patch**: A patch strategy that uses structural Kubernetes schema keys to merge partial YAML definitions into target manifests.
* **JSON Patch (RFC 6902)**: A standardized notation for applying explicit data modifications (`add`, `remove`, `replace`) directly to JSON/YAML fields.
* **Generator**: A built-in Kustomize module designed to dynamically construct Kubernetes primitives (`ConfigMap`, `Secret`) from environment variables, files, or key-value pairs.
* **Transformer**: A processing component that alters resource attributes in bulk across an entire manifest tree (e.g., modifying image tags or injecting labels).

---

## 9. Code Snippets & Configuration Examples

### Base Directory Setup

`kustomize/base/deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: nginx:1.21
        ports:
        - containerPort: 80
        envFrom:
        - configMapRef:
            name: app-config
```

`kustomize/base/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

configMapGenerator:
- name: app-config
  literals:
  - DB_HOST=base-db.internal
  - LOG_LEVEL=info
```

---

### Overlay Setup (Development)

`kustomize/overlays/dev/patch-replicas.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
```

`kustomize/overlays/dev/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- "../../base"

namespace: development

namePrefix: dev-

commonLabels:
  environment: development
  managed-by: kustomize

patches:
- path: patch-replicas.yaml

images:
- name: nginx
  newTag: 1.25-alpine

configMapGenerator:
- name: app-config
  behavior: merge
  literals:
  - LOG_LEVEL=debug
```

---

### Overlay Setup (Production)

`kustomize/overlays/prod/kustomization.yaml`:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- "../../base"

namespace: production

namePrefix: prod-

commonLabels:
  environment: production
  tier: critical

patches:
- target:
    kind: Deployment
    name: web-app
  patch: |-
    - op: replace
      path: /spec/replicas
      value: 10

images:
- name: nginx
  newName: enterprise-registry.example.com/nginx
  newTag: 1.25.3-hardened
```

---

### Operational Commands

```bash
# Preview rendered YAML output for Development
kustomize build kustomize/overlays/dev

# Apply Development environment directly to cluster using kubectl
kubectl apply -k kustomize/overlays/dev

# Preview Production manifest output
kubectl kustomize kustomize/overlays/prod

# Apply Production configuration directly
kubectl apply -k kustomize/overlays/prod

# Delete resources generated by an overlay
kubectl delete -k kustomize/overlays/dev
```

---

## 10. Best Practices

* **Keep Base Files Standard**: Base manifests should be valid, plain Kubernetes files that can run independently without requiring modifications.
* **DRY Overlays**: Avoid repeating configuration blocks across multiple overlays. Define shared values in the Base configuration and keep overlays focused on environment-specific overrides.
* **Use Hash Suffixes for ConfigMaps & Secrets**: Preserve default Kustomize hash appending behavior to ensure Pod deployments trigger rolling updates when configuration data changes.
* **GitOps Alignment**: Store Base and Overlay directories in Git repositories to enable declarative synchronization through tools like ArgoCD or Flux CD.
* **Use `images` Transformers for Tag Mutations**: Prefer using Kustomize's native `images` stanza over manual patches to update container image repositories and tags cleanly.

---

## 11. Common Mistakes

* **Mixing Raw `kubectl apply -f` with Kustomize Folders**: Running `kubectl apply -f` on a Kustomize directory processes raw target files without applying patches or resolving `kustomization.yaml`. Always use the `-k` flag (`kubectl apply -k`).
* **Modifying Base Directives for One Environment**: Editing base YAML files to suit a single environment breaks configuration reuse for all other overlays.
* **Incorrect Relative Paths**: Relative paths in `resources` or `patches` sections must accurately reflect directory structures (e.g., using `../../base` correctly).
* **Duplicate Resource Names Across Overlays**: Omitting prefixes like `namePrefix` or `namespace` directives when applying multiple overlays to a single cluster can lead to resource collisions.

---

## 12. Interview Questions

### Q1: How does Kustomize differ fundamentally from Helm?
**Ideal Answer:** Helm is a package manager that relies on string-replacement templating (using Go templates) to generate manifests from `values.yaml` files. Kustomize, by contrast, is a template-free configuration engine that reads valid standard Kubernetes YAML files and applies structural transformations, generators, and patches onto an Abstract Syntax Tree (AST). Additionally, Kustomize is built directly into `kubectl` via the `-k` flag.

### Q2: How does Kustomize ensure Pods update when a ConfigMap changes?
**Ideal Answer:** Kustomize uses content-based hashing via `configMapGenerator`. When generating a `ConfigMap`, Kustomize appends a hash derived from its content to the resource name (e.g., `app-config-7g8k2h9f4d`). It automatically updates references to that `ConfigMap` within Deployment manifests. When configuration content changes, the dynamic name changes as well, triggering a rolling deployment update in Kubernetes.

### Q3: What is the difference between Strategic Merge Patch and JSON Patch (RFC 6902) in Kustomize?
**Ideal Answer:** Strategic Merge Patch uses Kubernetes-aware target schema semantics to merge YAML structures based on primary target fields (e.g., matching container names in a array). JSON Patch (RFC 6902) provides path-specific mutations (`add`, `remove`, `replace`) using an explicit target path schema (e.g., `/spec/template/spec/containers/0/image`).

---

## 13. Certification Questions

### Question 1 (CKAD)
You are instructed to prepare a staging environment deployment using Kustomize. You must update the container tag for an image named `redis` to `6.2-alpine` without editing base manifest files directly. Which stanza inside `kustomization.yaml` is the standard method to accomplish this?

- A) `patchesStrategicMerge: - redis:6.2-alpine`
- B) `images: - name: redis \n newTag: 6.2-alpine`
- C) `transformers: - set-image: redis:6.2-alpine`
- D) `generators: - imageGenerator: redis:6.2-alpine`

**Correct Answer:** **B**
**Explanation:** The native `images` stanza in `kustomization.yaml` allows updating image tags or names without requiring custom JSON or Strategic Merge patches.

---

### Question 2 (CKA)
A developer executes `kubectl apply -f kustomize/overlays/dev/` and notices that `kustomization.yaml` is ignored and patches are not applied. What is the root cause?

- A) The `kustomization.yaml` file lacks valid root privileges.
- B) The `-f` flag processes raw files sequentially; Kustomize processing requires the `-k` flag (`kubectl apply -k`).
- C) The `kubectl` installation is missing the third-party Kustomize binary plugin.
- D) Relative references in `overlays/dev/kustomization.yaml` are broken.

**Correct Answer:** **B**
**Explanation:** The `-f` flag treats directory contents as standard raw manifests. To process Kustomize configurations, `kubectl` must be called with `-k`.

---

## 14. Real-World Examples

### Multi-Tenant GitOps Deployment with ArgoCD
A SaaS provider uses a core base configuration for an application service across 50 tenant environments. Instead of maintaining 50 separate Helm values files or raw manifest sets:
* The shared manifest lives in `git/base/`.
* Each tenant directory resides in `git/overlays/tenant-xx/`.
* Each overlay imports `../../base`, defines specific namespaces, injects tenant database credentials via `secretGenerator`, and sets localized CPU/Memory limits.
* ArgoCD monitors the overlay paths and deploys customized, tenant-isolated workloads automatically.

```
git-repo/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── tenant-alpha/
    │   └── kustomization.yaml
    └── tenant-beta/
        └── kustomization.yaml
```

---

## 15. Analogies

### The Overhead Transparency Film
Think of the Base manifest as a page in a textbook. An Overlay is a transparent sheet placed over that page. You write environment adjustments (such as adding annotations or changing values) directly onto the clear overlay. When viewed from above, you see a merged, context-specific result, while the original textbook page beneath remains unchanged.

### Standardized Modular Manufacturing
Consider a modular vehicle assembly line:
* **Base**: The standard chassis and core motor framing built for every car model.
* **Overlays**: Custom trim line packages (e.g., Sport Edition vs Off-Road Edition).
* **Kustomize Engine**: The assembly mechanism that combines standard framing specs with model-specific trim packages to build customized vehicles efficiently.

---

## 16. Frequently Asked Questions

### Can Kustomize be used without installing a separate binary?
Yes. Kustomize has been integrated directly into `kubectl` since version `1.14`. You can execute Kustomize commands natively using `kubectl apply -k <dir>` or `kubectl kustomize <dir>`.

### When should I choose Helm over Kustomize?
Helm is ideal when distributing third-party software packages (e.g., ingress controllers, databases) to external consumers who need configurable default values. Kustomize works best for internal applications and GitOps pipelines where you manage both source code and underlying deployment specifications directly.

### How do I override a ConfigMap created in the base directory within an overlay?
In your overlay's `kustomization.yaml`, define a `configMapGenerator` using the same name as the base and set `behavior: merge` or `behavior: replace`.

---

## 17. Related Technologies

* **Helm**: Kubernetes package manager using template interpolation.
* **Argo CD / Flux CD**: Declarative GitOps continuous delivery engines natively supporting Kustomize targets.
* **kpt**: Google-backed packaging tool for managing KRM (Kubernetes Resource Model) configurations using functions.
* **Skaffold**: Local Kubernetes development tool with built-in Kustomize build support.

---

## 18. Important Quotes

> *"Kustomize is a native Kubernetes configuration management tool that relies on declarative, template-free customization using plain YAML files."*

> *"In the base folder, you put files that remain constant across all target environments. Overlays layer environment-specific changes over top."*

> *"Every time you modify data in a configMapGenerator, Kustomize generates a new hash suffix. This updates deployment references and triggers rolling pod restarts automatically."*

---

## 19. Glossary

| Term | Category | Description |
| :--- | :--- | :--- |
| **AST** | Architecture | Abstract Syntax Tree: In-memory representation of structured document data used during transformations. |
| **Base** | Kustomize Core | A folder storing shared Kubernetes baseline manifests and a `kustomization.yaml` file. |
| **ConfigMapGenerator** | Module | Kustomize feature that dynamically creates ConfigMaps from files or literal string keys. |
| **JSON Patch** | RFC 6902 | Format defining targeted operations (`add`, `replace`, `delete`) on structural JSON data. |
| **KRM** | Specification | Kubernetes Resource Model: Declarative resource architecture used across Kubernetes objects. |
| **NamePrefix** | Transformer | Appends a string prefix to the `metadata.name` of all resources in a build context. |
| **Overlay** | Kustomize Core | A directory layering environment-specific changes over a designated Base. |
| **Strategic Merge Patch** | Patching | Kubernetes-native patch merging using structural schema definitions and target merge keys. |

---

## 20. One-Page Cheat Sheet

### Common `kustomization.yaml` Directives

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:               # Paths to raw manifests or base directories
- deployment.yaml
- ../../base

namespace: prod-env       # Injects namespace across all target resources

namePrefix: prod-        # Appends prefix to metadata.name attributes
nameSuffix: -v1          # Appends suffix to metadata.name attributes

commonLabels:            # Injects key-value labels across all resources
  app: payment-api

images:                  # Mutates container image tags & registries
- name: nginx
  newName: my-registry/nginx
  newTag: 1.25.3

patches:                 # Applies inline patches or patch file paths
- path: patch-replicas.yaml
```

### Key Commands

```bash
# Preview build output
kustomize build <path>
kubectl kustomize <path>

# Apply overlay configurations directly
kubectl apply -k <path>

# Delete overlay resource targets
kubectl delete -k <path>
```

---

## 21. Flash Cards

- **Card 1 | Kustomize Architecture**
  - **Q:** How does Kustomize handle configuration files without templating engines?
  - **A:** It reads plain Kubernetes YAML files into an Abstract Syntax Tree (AST) and applies structural patches and transformations over a Base configuration.

- **Card 2 | CLI Commands**
  - **Q:** What flag is required to run a Kustomize directory using `kubectl apply`?
  - **A:** The `-k` flag (e.g., `kubectl apply -k overlays/dev`).

- **Card 3 | Dynamic Resource Generation**
  - **Q:** Why does Kustomize append a content-hash suffix to generated ConfigMaps?
  - **A:** To update the deployment's resource reference when configuration changes occur, automatically triggering rolling pod restarts.

- **Card 4 | Directory Design**
  - **Q:** What is the primary operational distinction between a Base and an Overlay directory?
  - **A:** The Base contains shared, invariant resource specifications, while Overlays contain environment-specific configurations that patch the Base.

- **Card 5 | Transformers**
  - **Q:** Which directive updates container image tags without using explicit JSON patch statements?
  - **A:** The `images:` directive block inside `kustomization.yaml`.

---

## 22. Quiz

### Q1: What is the primary purpose of Kustomize in Kubernetes configuration management?
- A) Injecting Go templates into Helm charts at runtime
- B) Managing declarative configurations template-free using base and overlay patches
- C) Managing virtual machine image provisioning
- D) Intercepting API server requests as a dynamic admission controller  
**Correct Answer:** **B**  
**Explanation:** Kustomize enables template-free, declarative customization of Kubernetes manifests through base and overlay layers.

### Q2: Which command builds and previews rendered Kustomize manifests?
- A) `kubectl generate -f .`
- B) `kustomize build <directory>`
- C) `kubectl render -k .`
- D) `kustomize compile <directory>`  
**Correct Answer:** **B**  
**Explanation:** `kustomize build <directory>` (or `kubectl kustomize <directory>`) processes the target path and prints the rendered YAML to `stdout`.

### Q3: How are overlay directories linked back to base configurations in `kustomization.yaml`?
- A) Via the `import:` block
- B) Via the `resources:` block using relative directory paths
- C) Via the `bases:` stanza (deprecated in favor of `resources:`)
- D) Both B and C are functionally valid  
**Correct Answer:** **D**  
**Explanation:** Historically `bases:` was used, but modern versions handle both external bases and internal manifests within the `resources:` array.

### Q4: How does Kustomize trigger a rolling deployment restart when a ConfigMap updates?
- A) By calling `kubectl rollout restart` internally
- B) By modifying deployment annotations with time-stamps
- C) By appending dynamic content hashes to generated `ConfigMap` names
- D) By restarting the kubelet daemon automatically  
**Correct Answer:** **C**  
**Explanation:** Changing `ConfigMap` contents alters the output hash suffix, updating deployment volume and environment specifications to trigger rolling updates.

### Q5: What patch format uses JSON target fields (`op`, `path`, `value`) to modify resources?
- A) Strategic Merge Patch
- B) Go Template Mutation
- C) JSON Patch (RFC 6902)
- D) YAML Anchors  
**Correct Answer:** **C**  
**Explanation:** RFC 6902 JSON Patches use structured operation lists (`add`, `remove`, `replace`) targeted at precise JSON field paths.

### Q6: What happens if you run `kubectl apply -f` on a Kustomize overlay directory?
- A) `kubectl` auto-detects `kustomization.yaml` and processes it correctly
- B) `kubectl` attempts to parse files as raw resources, ignoring patches and transformer rules
- C) The deployment fails immediately with an API schema error
- D) `kubectl` automatically installs the Helm engine  
**Correct Answer:** **B**  
**Explanation:** The `-f` flag processes raw files sequentially without evaluating Kustomize patch instructions.

### Q7: Which directive in `kustomization.yaml` automatically injects a label across all managed manifests?
- A) `commonLabels:`
- B) `globalMetadata:`
- C) `setLabels:`
- D) `appendLabels:`  
**Correct Answer:** **A**  
**Explanation:** `commonLabels:` injects defined key-value labels across all resource definitions, selectors, and metadata specifications.

### Q8: Can a base directory point to a remote Git repository?
- A) No, base paths must exist on the local file system
- B) Yes, Kustomize can reference remote Git repo URLs directly in the `resources:` list
- C) Only if the repository is converted into a compressed `.tar.gz` archive
- D) Only when using enterprise Kustomize extensions  
**Correct Answer:** **B**  
**Explanation:** Kustomize natively supports fetching remote base targets directly using Git repository reference URLs.

### Q9: Which directive updates image tags dynamically without raw file edits?
- A) `tagMap:`
- B) `containerRegistry:`
- C) `images:`
- D) `imagePatcher:`  
**Correct Answer:** **C**  
**Explanation:** The `images:` stanza target allows modifying image names, registries, and tags across configurations cleanly.

### Q10: How can you merge additional key-value literals into an existing base ConfigMap inside an overlay?
- A) Set `behavior: merge` in the overlay's `configMapGenerator` definition
- B) Define `mode: append` inside the deployment file
- C) Use standard Linux `cat` redirection commands
- D) ConfigMaps cannot be modified once defined in a base directory  
**Correct Answer:** **A**  
**Explanation:** Setting `behavior: merge` (or `behavior: replace`) in a generator overlay allows modifying or extending base resource mappings.

---

## 23. Action Items

* [ ] Install `kustomize` locally or verify `kubectl` version is `1.14+`.
* [ ] Clone the video repository: `git clone https://github.com/iam-veeramalla/kustomize-zero-to-hero`.
* [ ] Create a `base/` directory containing sample deployment and service manifests alongside a root `kustomization.yaml`.
* [ ] Build two overlay environments (`overlays/dev/` and `overlays/prod/`) that import `../../base`.
* [ ] Add a replica scaling patch (`patch-replicas.yaml`) to the dev overlay and scale production using JSON patching.
* [ ] Test generated outputs locally using `kustomize build overlays/dev` and `kustomize build overlays/prod`.
* [ ] Deploy configurations to a target test cluster using `kubectl apply -k overlays/dev`.

---

## 24. Recommended Further Reading

* **Official Kustomize Documentation**: https://kubectl.docs.kubernetes.io/references/kustomize/
* **GitHub Demonstration Repository**: https://github.com/iam-veeramalla/kustomize-zero-to-hero
* **Kubernetes Declarative Management**: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/
* **RFC 6902 (JavaScript Object Notation Patching)**: https://datatracker.ietf.org/doc/html/rfc6902