# Argo CD — Complete Study Notes

**Topic:** GitOps Continuous Delivery for Kubernetes
**Includes:** Concepts, Architecture, Benefits, Multi-Cluster Strategy, and a Full Hands-On Demo

---

## Table of Contents

1. [Prerequisite — What Is GitOps](#1-prerequisite--what-is-gitops)
2. [What Is Argo CD](#2-what-is-argo-cd)
3. [The CD Workflow Without Argo CD — and Its Three Problems](#3-the-cd-workflow-without-argo-cd--and-its-three-problems)
   - [3.1 The Typical Flow](#31-the-typical-flow)
   - [3.2 Problem 1 — Tooling Setup Burden](#32-problem-1--tooling-setup-burden)
   - [3.3 Problem 2 — Credentials Sprawl and Security Risk](#33-problem-2--credentials-sprawl-and-security-risk)
   - [3.4 Problem 3 — No Visibility Into Deployment Status](#34-problem-3--no-visibility-into-deployment-status-the-most-important-one)
   - [3.5 The Conclusion](#35-the-conclusion)
4. [The CD Workflow With Argo CD — Reversing the Flow](#4-the-cd-workflow-with-argo-cd--reversing-the-flow)
   - [4.1 The Core Idea](#41-the-core-idea)
   - [4.2 The Full Workflow, Step by Step](#42-the-full-workflow-step-by-step)
   - [4.3 What Argo CD Can Read](#43-what-argo-cd-can-read)
   - [4.4 The Result — Separation of Concerns](#44-the-result--separation-of-concerns)
5. [Repository Best Practice — Separating Source Code from Config](#5-repository-best-practice--separating-source-code-from-config)
   - [5.1 Reason 1 — Config Is More Than the Deployment File](#51-reason-1--config-is-more-than-the-deployment-file)
   - [5.2 Reason 2 — Avoid Pointless CI Runs](#52-reason-2--avoid-pointless-ci-runs)
   - [5.3 Reason 3 — Avoid Complex Pipeline Logic](#53-reason-3--avoid-complex-pipeline-logic)
6. [Benefits of GitOps with Argo CD](#6-benefits-of-gitops-with-argo-cd)
   - [6.1 Git as the Single Source of Truth](#61-benefit--git-as-the-single-source-of-truth)
   - [6.2 The Manual-Change Problem — and Self-Healing](#62-the-manual-change-problem--and-self-healing)
   - [6.3 The Escape Hatch — Alert Instead of Override](#63-the-escape-hatch--alert-instead-of-override)
   - [6.4 Version Control, History, and Audit Trail](#64-benefit--version-control-history-and-audit-trail)
   - [6.5 Easy Rollback](#65-benefit--easy-rollback)
   - [6.6 Cluster Disaster Recovery](#66-benefit--cluster-disaster-recovery)
7. [Benefits Specific to Argo CD](#7-benefits-specific-to-argo-cd)
   - [7.1 Kubernetes Access Control Through Git](#71-kubernetes-access-control-through-git)
   - [7.2 Access Management for Non-Human Users](#72-access-management-for-non-human-users)
8. [Argo CD as a Kubernetes Extension](#8-argo-cd-as-a-kubernetes-extension)
   - [8.1 The Main Benefit — Real Visibility](#81-the-main-benefit--real-visibility)
9. [The Big Picture — Desired vs. Actual State](#9-the-big-picture--desired-vs-actual-state)
10. [How to Configure Argo CD](#10-how-to-configure-argo-cd)
    - [10.1 Deployment](#101-deployment)
    - [10.2 The Application CRD](#102-the-application-crd)
    - [10.3 The AppProject CRD](#103-the-appproject-crd)
11. [Working with Multiple Clusters](#11-working-with-multiple-clusters)
    - [11.1 Case A — Multiple Cluster Replicas, One Environment](#111-case-a--multiple-cluster-replicas-one-environment)
    - [11.2 Case B — Multiple Environments](#112-case-b--multiple-environments)
    - [11.3 Two Options for Achieving Promotion](#113-two-options-for-achieving-promotion)
12. [Does Argo CD Replace Jenkins or GitLab CI/CD?](#12-does-argo-cd-replace-jenkins-or-gitlab-cicd)
    - [12.1 Alternatives](#121-alternatives)
13. [Hands-On Demo — Setup and Overview](#13-hands-on-demo--setup-and-overview)
    - [13.1 Starting Materials](#131-starting-materials)
    - [13.2 Repository Layout](#132-repository-layout)
    - [13.3 Plan](#133-plan)
14. [Hands-On Demo — Step-by-Step Walkthrough](#14-hands-on-demo--step-by-step-walkthrough)
    - [Step 1 — Install Argo CD](#step-1--install-argo-cd)
    - [Step 2 — Access the Argo CD UI](#step-2--access-the-argo-cd-ui)
    - [Step 3 — Log In](#step-3--log-in)
    - [Step 4 — Write the Application Configuration](#step-4--write-the-application-configuration)
    - [Step 5 — Understanding Every Field](#step-5--understanding-every-field)
    - [Step 6 — Understanding the Sync Interval](#step-6--understanding-the-sync-interval)
    - [Step 7 — Why Are These Options Off by Default?](#step-7--why-are-these-options-off-by-default)
    - [Step 8 — Apply the Configuration](#step-8--apply-the-configuration)
    - [Step 9 — Explore the Argo CD UI](#step-9--explore-the-argo-cd-ui)
    - [Step 10 — Test 1: Automatic Sync](#step-10--test-1-automatic-sync)
    - [Step 11 — Test 2: Pruning](#step-11--test-2-pruning)
    - [Step 12 — Test 3: Self-Healing](#step-12--test-3-self-healing)
    - [Step 13 — Manual Sync Trigger](#step-13--manual-sync-trigger)
15. [Command Reference](#15-command-reference)
16. [Key Takeaways](#16-key-takeaways)

---

## 1. Prerequisite — What Is GitOps

Argo CD is a **GitOps tool** gaining significant popularity in the DevOps world. Understanding GitOps first makes everything that follows far clearer.

The video is structured in three parts:

1. **What** Argo CD is, and the common use cases — *why* we need it
2. **How** Argo CD actually works and does its job
3. A **hands-on demo project** deploying Argo CD and setting up a fully automated CD pipeline for Kubernetes configuration changes

[↑ Back to top](#table-of-contents)

---

## 2. What Is Argo CD

As the name implies, Argo CD is a **Continuous Delivery tool**.

To understand it properly, first understand how continuous delivery is implemented in most projects using common tools like **Jenkins** or **GitLab CI/CD** — then compare.

**The questions this comparison answers:**

- Is Argo CD just another CD tool, or is something genuinely special about it?
- If it is special, does it **replace** established tools like Jenkins or GitLab CI/CD?

**Core positioning:** Argo CD was **purpose-built for Kubernetes**, following **GitOps principles**.

[↑ Back to top](#table-of-contents)

---

## 3. The CD Workflow Without Argo CD — and Its Three Problems

### 3.1 The Typical Flow

**Scenario:** A microservices application running in a Kubernetes cluster.
<img width="605" height="240" alt="image" src="https://github.com/user-attachments/assets/04b2b070-7f9e-414d-b36d-8ff35cb7acd0" />

<img width="1174" height="432" alt="image" src="https://github.com/user-attachments/assets/497643b4-c819-48bc-a97a-a998701fad6d" />


In most projects, steps 4 and 5 are simply a **continuation of the CI pipeline**. This is how many projects are set up — and it works, but it carries three real challenges.


### 3.2 Problem 1 — Tooling Setup Burden

You must **install and configure** tools like `kubectl`, `helm`, and similar **on the build automation server (Jenkins)** just so it can access the cluster and execute changes.

That is ongoing installation and configuration effort on a machine whose real job is building code.

### 3.3 Problem 2 — Credentials Sprawl and Security Risk

`kubectl` is only the Kubernetes **client** — to connect to a cluster it must be given **credentials**.

**What that means in practice:**

- You must configure **Kubernetes cluster credentials in Jenkins**
- If you use **EKS** (managed Kubernetes on AWS), you must **additionally** add **AWS credentials** to Jenkins

**Why this is more than a config chore — it's a security challenge:**

> You are handing your **cluster credentials to external services and tools.**

**And it scales badly:**

- **50 projects** deploying to the cluster → each application needs **its own Kubernetes credentials**, scoped so it can only access that specific application's resources
- **50 clusters** → the same configuration must be repeated **for every single cluster**

### 3.4 Problem 3 — No Visibility Into Deployment Status (the most important one)
<img width="1184" height="474" alt="image" src="https://github.com/user-attachments/assets/8a5c282a-021c-4df2-b5b8-753537849741" />


Once Jenkins deploys the application or applies any change to Kubernetes, it has **no further visibility into the deployment status.**

After `kubectl apply` executes, Jenkins does **not** know:

- Did the application actually get created?
- Is it in a **healthy** status?
- Is it **failing to start**?

You can only find out through **additional test steps** written afterward.

### 3.5 The Conclusion

The **CD portion** of the pipeline — specifically when working with Kubernetes — can be made **more efficient**. Argo CD was built for exactly this use case.
<img width="1071" height="469" alt="image" src="https://github.com/user-attachments/assets/a854945a-2ce7-4abc-bd22-eb6f261575e2" />


[↑ Back to top](#table-of-contents)

---

## 4. The CD Workflow With Argo CD — Reversing the Flow

### 4.1 The Core Idea

> **We simply reverse the flow.**

- **Instead of** externally accessing the cluster from a CI/CD tool like Jenkins…
- …**the tool is itself part of the cluster**
- **Instead of pushing** changes into the cluster, we use a **pull workflow**, where an agent inside the cluster — Argo CD — **pulls** those changes and **applies** them

This is **push model → pull model.**

### 4.2 The Full Workflow, Step by Step

**Setup (done once):**

1. **Deploy Argo CD** into the cluster
2. **Configure Argo CD:** "Connect to this Git repository and watch it for changes. If anything changes, automatically pull those changes and apply them in the cluster."

**Ongoing flow:**

3. Developers **commit application changes** to the **application source code repository**
4. The **Jenkins CI pipeline** automatically starts a build: **test → build image → push image to repository**
5. Jenkins **updates the Kubernetes manifest file** (e.g. `deployment.yaml`) with the **new image version** — in the **separate configuration repository**
6. As soon as that configuration file changes in Git, **Argo CD immediately knows**, because it was configured to constantly monitor the repository
7. Argo CD **pulls and applies** those changes in the cluster **automatically**

### 4.3 What Argo CD Can Read

Argo CD supports Kubernetes manifests defined as:

- **Plain YAML files**
- **Helm charts**
- **Kustomize files**
- **Other template files**

All of these eventually get generated into **plain Kubernetes YAML files**, so you can use any of them with Argo CD.

### 4.4 The Result — Separation of Concerns

The configuration repository tracked and synced by Argo CD is sometimes called the **GitOps repository**.

Whether that repository is updated by **Jenkins** (image version bump in `deployment.yaml`) or by **DevOps engineers** (changing other manifest files), the changes get **automatically pulled and applied** by Argo CD.

**We end up with two separate pipelines:**

- **CI pipeline** — owned mostly by **developers**, configured on **Jenkins**
- **CD pipeline** — owned by **operations / DevOps teams**, configured using **Argo CD**

> You still get a fully automated CI/CD pipeline, but with **separation of concerns** — different teams responsible for different parts of the full pipeline.

[↑ Back to top](#table-of-contents)

---

## 5. Repository Best Practice — Separating Source Code from Config

> **It has been established as a best practice to have separate repositories for application source code and application configuration code.**

Kubernetes manifest files for the application live in **their own Git repository.**

### 5.1 Reason 1 — Config Is More Than the Deployment File

Application configuration code includes not only the deployment file but also:

- ConfigMap
- Secret
- Service
- Ingress
- …essentially **everything the application needs to run in the cluster**

These files can **change completely independently** of the source code.

### 5.2 Reason 2 — Avoid Pointless CI Runs

When you update a **Service YAML** file — pure configuration, not code — you **don't want to run the whole CI pipeline**, because **the code itself hasn't changed.**

### 5.3 Reason 3 — Avoid Complex Pipeline Logic

You also **don't want complex logic in your build pipeline** that has to decide and check *what actually changed.* Separating the repositories removes that problem entirely.

[↑ Back to top](#table-of-contents)

---

## 6. Benefits of GitOps with Argo CD

> **Important framing:** These are **GitOps principles and benefits** — the benefits you get from implementing these principles with *whatever* GitOps tool you choose. They are **not** benefits unique to Argo CD; Argo CD simply helps you implement them.

### 6.1 Benefit — Git as the Single Source of Truth

The **whole Kubernetes configuration is defined as code in a Git repository.**

Instead of everyone working from their own laptops, executing scripts, running `kubectl apply` or `helm install`, **everyone uses the same interface** to make changes to the cluster.

### 6.2 The Manual-Change Problem — and Self-Healing

**The tempting shortcut:** Someone thinks, *"Let me just quickly update something in the cluster — a `kubectl apply` is much faster than writing the change, committing, pushing, getting a review from colleagues, and eventually merging."*

**What Argo CD does about it:**

Argo CD **watches two things**:

1. Changes in the **Git repository**
2. Changes in the **cluster**

Any time a change happens in **either place**, Argo CD **compares the two states.**

- **Desired state** = what's defined in the Git repository
- **Actual state** = what's live in the cluster

If they **no longer match**, Argo CD knows it must act — and its action is **always the same**: **sync whatever is defined in Git into the cluster.**

**So if someone makes a manual change:** Argo CD detects the divergence and **overwrites whatever was done manually.**

**Why this matters:**

- It **guarantees** the Git repository remains, at all times, the **only source of truth** for cluster state
- It gives you **full transparency** — whatever is defined in that repository *is* exactly the state of your cluster

### 6.3 The Escape Hatch — Alert Instead of Override

Some projects need time to adjust to this workflow, and sometimes engineers genuinely need a **quick way to update the cluster** before changing the code.

Argo CD can be configured to **not** automatically override and undo manual cluster changes, and **instead send out an alert** that something was changed manually and needs to be reflected in the code.

### 6.4 Benefit — Version Control, History, and Audit Trail

Because Git is the single interface for change — instead of **untrackable `kubectl apply` commands** — you get:

- Every change **documented in a version-controlled way**
- A **history of changes**
- An **audit trail** of **who changed what** in the cluster
- A way for teams to **collaborate**: propose a change to Kubernetes, let others **discuss and work on it**, then **merge it into the main branch**

### 6.5 Benefit — Easy Rollback

Argo CD pulls changes and applies them. If something **breaks** — say a new application version **fails to start** — you simply **revert to the previous working state in Git history**.

**Why this scales:** With **thousands of clusters** all updated from the same Git repository, this is dramatically more efficient. You **don't** have to manually revert every component with `kubectl delete` or `helm uninstall` and clean everything up.

> You simply **declare the previous working state**, and the cluster is **synced back to it.**

### 6.6 Benefit — Cluster Disaster Recovery

**Scenario:** You have an EKS cluster in region 1A, and that cluster **completely crashes.**

**Recovery with GitOps:**

1. **Create a new cluster**
2. **Point it at the Git repository** where the complete cluster configuration is defined
3. It **recreates the exact same state** as the previous one — **without any intervention** from you

This works because the **whole cluster is described in code, declaratively.**

[↑ Back to top](#table-of-contents)

---

## 7. Benefits Specific to Argo CD

### 7.1 Kubernetes Access Control Through Git

You don't want every team member able to change the cluster — **especially in production.**

Using Git, you configure access rules easily:

- **Every member** of the DevOps / operations team — **including junior engineers** — can **initiate or propose** any change and open **pull requests**
- Only a **handful of senior engineers** can **approve and merge** those requests

**The result:** a clean way of managing **cluster permissions indirectly through Git**, without having to create **cluster roles and users for every team member** with different access permissions.

> **Easy cluster access management** — and because of the **pull model**, you only need to give engineers access to the **Git repository**, not the cluster directly.

### 7.2 Access Management for Non-Human Users

The same benefit applies to **non-human users** such as CI/CD tools like Jenkins.

- You **don't need to give external cluster access to Jenkins** or other external tools
- Argo CD is **already running in the cluster** and is **the only thing that applies changes**

> **Cluster credentials no longer have to exist outside the cluster**, because the agent runs **inside** it.

This makes **managing security across all your Kubernetes clusters dramatically easier.**

[↑ Back to top](#table-of-contents)

---

## 8. Argo CD as a Kubernetes Extension

Argo CD is deployed **directly in the Kubernetes cluster** — but that alone isn't special, since you can also deploy Jenkins in a cluster.

> The real point: Argo CD is an **extension to the Kubernetes API.**

**How it works:** Argo CD **leverages Kubernetes resources themselves.** Instead of building all functionality from scratch, it **reuses existing Kubernetes functionality**:

- **etcd** — for storing its data
- **Kubernetes controllers** — for monitoring and comparing actual vs. desired state

### 8.1 The Main Benefit — Real Visibility

This gives Argo CD **visibility inside the cluster that Jenkins simply does not have**, allowing it to provide **real-time updates of the application state.**

Argo CD can monitor deployment state **after** the application is deployed or **after** configuration changes are made. For example, when you deploy a new application version, you can watch **in real time in the Argo CD UI**:

- Configuration files were **applied**
- Pods were **created**
- The application is **running and healthy** — or is **failing** and a **rollback is needed**

[↑ Back to top](#table-of-contents)

---

## 9. The Big Picture — Desired vs. Actual State

Zooming out, there are three elements:

- **Git repository** — the **desired state** of the cluster, at any time
- **Kubernetes cluster** — the **actual state**
- **Argo CD** — the **agent in the middle**

> Argo CD's entire job: **make sure these two are always in sync**, updating the **actual state** with the **desired state** as soon as they diverge.

[↑ Back to top](#table-of-contents)

---

## 10. How to Configure Argo CD

### 10.1 Deployment

You deploy Argo CD into Kubernetes **just like you deploy Prometheus, Istio, or any other tool.**

Because it was **purpose-built for Kubernetes**, it **extends the Kubernetes API with CRDs** (Custom Resource Definitions), which lets you configure Argo CD using **Kubernetes-native YAML files.**

### 10.2 The Application CRD

The **main component** of Argo CD is the **Application**.

The core of an Application definition is: **which Git repository should be synced with which Kubernetes cluster.**

- The Git repository can be **any** Git repository
- The Kubernetes cluster can be **the cluster Argo CD itself runs in**, *or* an **external cluster** that Argo CD manages

You can configure **multiple Applications** — for example, one per microservice.

### 10.3 The AppProject CRD

If some applications **belong together**, you can **group them** in another CRD called **AppProject**.

[↑ Back to top](#table-of-contents)

---

## 11. Working with Multiple Clusters

### 11.1 Case A — Multiple Cluster Replicas, One Environment

**Scenario:** Three cluster replicas for a **dev environment**, in **three different regions**.

**Setup:** Argo CD runs in **one** of the clusters, configured to deploy changes to **all three replicas at the same time.**

**Benefit:** Kubernetes administrators configure and manage **only one Argo CD instance**, and that single instance configures a **fleet of clusters** — whether that's **three clusters in three regions** or **a thousand cluster replicas distributed worldwide.**

### 11.2 Case B — Multiple Environments

**Scenario:** Development, Staging, and Production environments — each possibly with their own cluster replicas.

**Setup:** Each environment may run **its own Argo CD instance**, but there is still **one repository** where cluster configuration is defined.

**The requirement:** You **don't** want to deploy the same configuration to all environments at once. Changes must be **tested on each environment first, then promoted to the next**:

> Apply to **Development** → if successful, promote to **Staging** → and so on.

### 11.3 Two Options for Achieving Promotion

**Option 1 — Multiple Git branches per environment**
Separate `development`, `staging`, and `production` branches.
*Assessment:* **Probably not the best option**, even though it is commonly used.

**Option 2 — Overlays with Kustomize (the better option)**
You have **your own context for each environment.** Overlays let you **reuse the same base YAML files** and **selectively change specific parts** for different environments.

**How it works in practice:**

- The **development** CI pipeline updates the template in the **development overlay**
- The **staging** CI pipeline updates the template in the **staging overlay**
- …and so on

This lets you **automate and streamline applying changes across environments.**

[↑ Back to top](#table-of-contents)

---

## 12. Does Argo CD Replace Jenkins or GitLab CI/CD?

**Short answer: Not really.**

**Why:**

- You **still need the CI pipeline** — you must still configure pipelines to **test and build application code changes**
- Argo CD, as the name implies, is a **replacement for the CD pipeline** — but **specifically for Kubernetes**
- For **other platforms**, you will still need a different tool

### 12.1 Alternatives

Argo CD is **not the only GitOps CD tool** for Kubernetes. Many alternatives already exist, and as the trend grows, more will emerge.

- **Flux CD** — currently one of the most popular, working with and implementing **the same GitOps principles**

[↑ Back to top](#table-of-contents)

---

## 13. Hands-On Demo — Setup and Overview

**Goal:** Set up a **fully automated CD pipeline** for Kubernetes configuration changes with Argo CD.

### 13.1 Starting Materials

- A **Git repository** containing `deployment.yaml` and `service.yaml`, where the deployment references the demo application **image version 1.0**
- **Three image versions** already built and available in a **public Docker repository** (so you can follow along)
- An **empty Minikube cluster** — though any Kubernetes cluster works

*(We simply imagine some CI pipeline built those images; for this demo it doesn't matter — they just exist.)*

### 13.2 Repository Layout

- A **`dev` folder** containing:
  - `deployment.yaml` — **two replicas**, image pointing at the public demo app **version 1.0**
  - `service.yaml` — a simple **internal service**

There is **also a separate application source-code repository** used to build the image — reinforcing the **separate repos** best practice. The source code itself is not relevant here; we only care about how the deployment works.

### 13.3 Plan

1. **Install Argo CD** in the Kubernetes cluster
2. **Configure Argo CD** with an **Application CRD** telling it: *"This is the Git repository you must track and sync with this Minikube cluster."*
3. From then on, **do nothing in Kubernetes directly** — only update config files in Git, and Argo CD pulls and applies them automatically

[↑ Back to top](#table-of-contents)

---

## 14. Hands-On Demo — Step-by-Step Walkthrough

### Step 1 — Install Argo CD

Installing Argo CD is **essentially a one-liner**, per the official Getting Started guide.

**Create the namespace** (all Argo CD components run here):

```bash
kubectl create namespace argocd
```

**Apply the install manifest:**

```bash
kubectl apply -n argocd -f <argocd-install.yaml>
```

> **Tip:** As an alternative, **download the YAML file first** so you can inspect and save it, and know exactly what will be deployed.

**Verify:**

```bash
kubectl get pod -n argocd
```

Wait until **all pods are running** before proceeding.

### Step 2 — Access the Argo CD UI

**Find the service:**

```bash
kubectl get svc -n argocd
```

You'll see **`argocd-server`**, accessible on **HTTP and HTTPS** ports.

**Port-forward it locally:**

```bash
kubectl port-forward -n argocd svc/argocd-server 8080:443
```

This forwards service requests to **localhost port 8080**.

**Open the URL.** You'll get a **connection warning** because it's HTTPS but not trusted — proceed anyway.

### Step 3 — Log In

- **Username:** `admin`
- **Password:** **auto-generated** and stored in a secret called **`argocd-initial-admin-secret`**

**Retrieve it:**

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o yaml
```

The `password` attribute holds a **base64-encoded** value. **Decode it**, ignore any trailing percentage sign, copy the string, and sign in.

**At this point Argo CD is empty** — no applications, because it hasn't been configured yet.

### Step 4 — Write the Application Configuration

Clone the configuration repository locally and create **`application.yaml`** inside it. *(The Argo CD config lives in the same configuration repository.)*

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default

  source:
    repoURL: <your-git-repo-url>
    targetRevision: HEAD
    path: dev

  destination:
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
      - CreateNamespace=true
    automated:
      selfHeal: true
      prune: true
```

### Step 5 — Understanding Every Field

**`apiVersion: argoproj.io/v1alpha1`**
For custom resource definitions, the API version comes **from the project itself**.

> **Important:** This **changes over time** as the API moves from alpha → beta → stable release. **Always refer to the documentation** to confirm the correct value when creating an Application.

**`kind: Application`** — the main Argo CD component

**`metadata.namespace: argocd`** — the Application component is created in the **same namespace where Argo CD runs**

**`spec.project: default`** — you can group multiple Applications into a project; if you don't care, use `default`, where everything goes by default

**The two things configured in every Application:**

**1. `source` — the Git repository Argo CD connects to and syncs**

- **`repoURL`** — the Git repository URL
- **`targetRevision: HEAD`** — always the **last commit** in that repository
- **`path: dev`** — lets you sync/track only a **specific path** in the repository

**2. `destination` — the cluster where Argo CD applies what it found**

- **`server: https://kubernetes.default.svc`** — the **Kubernetes API server** address. Because Argo CD runs **inside** the destination cluster, you can use the **internal DNS name** of the API server rather than an external endpoint.
  - Verify with `kubectl get svc` in the `default` namespace — you'll see the `kubernetes` service, the internal service for the API server
  - For **external clusters** or **multi-cluster sync**, you would instead put the **external address** of the cluster here
- **`namespace: myapp`** — which namespace Argo CD should apply the configuration into. Only needed if it isn't `default`.

**`syncPolicy` — the automation switches**

**`syncOptions: - CreateNamespace=true`**
The `myapp` namespace doesn't exist yet. **By default this is `false`** — Argo CD will *not* auto-create a missing namespace. Setting it to `true` makes Argo CD create it automatically.

**`automated:`** — enables automatic syncing of Git changes.

> **By default automatic sync is turned off** — if you push something to Git, Argo CD does **not** automatically fetch it. The `automated` attribute enables it.

**`automated.selfHeal: true`**
If someone runs `kubectl apply` manually in the cluster, Argo CD **overrides it** and syncs the cluster back to the Git repository state.

**`automated.prune: true`**
If you **rename a component** (so the old name no longer exists) or **delete a whole YAML file**, Argo CD will also **delete that old component from the cluster.**

### Step 6 — Understanding the Sync Interval

The `automated` attribute configures Argo CD to poll the Git repository **every three minutes.**

> Argo CD does **not** pull changes the instant they happen — it **checks every three minutes**, then pulls and applies whatever changed.

**If you need instant sync:** configure a **webhook integration** between the Git repository and Argo CD.

**This demo uses the default** — regular interval-based checking.

### Step 7 — Why Are These Options Off by Default?

You must explicitly enable automatic sync, self-healing, and pruning. The likely reasons:

1. A **protection mechanism** — so something doesn't get **accidentally deleted**; you must opt in deliberately
2. Your **team may need time to transition** to this workflow, so automatic syncs and self-healing shouldn't be forced on you

Either way, **enabling them is not difficult**, and doing so **fully automates all these actions.**

> Note: within the **same cluster**, you may create **multiple such Applications** — for different namespaces or different environments.

### Step 8 — Apply the Configuration

**First, push `application.yaml` to the remote repository** — Argo CD connects to the *remote* repo, so the config should live there too.

**Then apply it to the cluster:**

```bash
kubectl apply -f application.yaml
```

> **This should ideally be the only `kubectl apply` you ever run in this project** — everything after this is auto-synchronized.

### Step 9 — Explore the Argo CD UI

Back in the UI, you'll now see **`myapp-argo-application`**, which has **successfully synced** everything from the repository.

**Click in for details.** A particularly useful feature is the **component overview graph**, showing everything the Application triggered:

- The **Argo CD Application** component itself
- The **Service** (with its service icon)
- The **Deployment**
- The **ReplicaSet** behind that deployment
- The **two Pod replicas** (matching the two replicas defined for the image)

**Click any component for more detail.** Clicking a **Pod** shows:

- Main data, **including the image**
- The **live-state manifest**
- **Events** for that pod
- **Logs**

**Clicking the Application** shows the details you provided — repository URL, `CreateNamespace` enabled, sync policy, prune, self-heal — all **editable in the UI**, plus a **manifest / YAML view** of the configuration.

### Step 10 — Test 1: Automatic Sync

**Action:** Edit `deployment.yaml` **directly in the GitLab UI** (no local commit/push needed) and change the image tag from **`1.0` → `1.2`**. Commit.

**Result:** At the next check interval, Argo CD sees the desired state is now `1.2` while the cluster runs `1.0`, and syncs automatically.

**Observed behavior:**

- The application shows **"Out of Sync"** very briefly
- The **Deployment updates**
- A **new ReplicaSet** is created in the background, which starts **two new pods**
- Verifying: pods now run image tag **`1.2`**, and the Deployment's desired state is **version 1.2**

### Step 11 — Test 2: Pruning

**Action:** **Rename** the deployment from `myapp-deployment` to `myapp`. Commit.

**Expected:** Because the resource with the old name no longer exists in Git, Argo CD should delete the old component in the cluster.

**Result:** Argo CD **removed the old `myapp-deployment`** and **created a new one** with its own ReplicaSet and two pods.

> **Pruning works.**

### Step 12 — Test 3: Self-Healing

**Action:** Change the cluster **directly**, bypassing Git:

```bash
kubectl edit deployment myapp -n myapp
```

Change replicas from **2 → 4** and save.

**Result:**

- **Two pods were added**
- **Argo CD immediately reverted it back to two replicas**

Re-running `kubectl edit deployment` confirms it says **two replicas** again.

> **The manual change was reverted — because `selfHeal: true` undoes any manual cluster change.**

### Step 13 — Manual Sync Trigger

If you're testing and **don't want to wait** for the interval, trigger synchronization manually in the UI:

1. **Refresh** — tells Argo CD to **compare** the cluster state with the repository state
2. **Sync** — actually **synchronizes** those states

[↑ Back to top](#table-of-contents)

---

## 15. Command Reference

| Command | Purpose |
| --- | --- |
| `kubectl create namespace argocd` | Create the namespace for Argo CD components |
| `kubectl apply -n argocd -f <install.yaml>` | Install all Argo CD components |
| `kubectl get pod -n argocd` | Verify all Argo CD pods are running |
| `kubectl get svc -n argocd` | Find the `argocd-server` service |
| `kubectl port-forward -n argocd svc/argocd-server 8080:443` | Expose the UI on localhost:8080 |
| `kubectl get secret argocd-initial-admin-secret -n argocd -o yaml` | Retrieve the base64-encoded admin password |
| `kubectl apply -f application.yaml` | Register the Application CRD (ideally the last manual apply) |
| `kubectl get svc` (default namespace) | Confirm the internal `kubernetes` API server service |
| `kubectl edit deployment <name> -n <namespace>` | Used only to demonstrate that self-heal reverts manual edits |

[↑ Back to top](#table-of-contents)

---

## 16. Key Takeaways

1. **The push model is the problem.** Giving Jenkins cluster credentials means tooling overhead, credential sprawl across projects and clusters, and zero deployment visibility.
2. **Argo CD reverses the direction.** An agent inside the cluster **pulls** from Git instead of an external tool **pushing** in. Cluster credentials never leave the cluster.
3. **Separate your repositories.** Application source code and application configuration code belong in different repos — config changes independently, and you don't want to trigger full CI builds or write branching logic to detect what changed.
4. **Git is the single source of truth.** Argo CD watches **both** Git and the cluster, compares states, and **always syncs toward Git** — overwriting manual changes, or alerting instead if you configure it that way.
5. **CI and CD become cleanly separated.** Developers own CI in Jenkins; DevOps owns CD in Argo CD. Still fully automated, but with clear ownership.
6. **Rollback and disaster recovery become trivial.** Revert a commit to roll back across thousands of clusters. Point a brand-new cluster at the repo to fully recreate a crashed one — no manual intervention.
7. **Access control moves into Git.** Juniors propose via pull requests; seniors approve and merge. No per-person cluster roles required; engineers need Git access, not cluster access.
8. **Argo CD extends the Kubernetes API.** It reuses etcd for storage and Kubernetes controllers for state comparison — which is precisely what gives it the real-time in-cluster visibility Jenkins lacks.
9. **Automation is opt-in.** `automated`, `selfHeal`, `prune`, and `CreateNamespace` are all **off by default** — deliberately, as a safety mechanism and to ease team transition.
10. **Default sync is every three minutes.** Use a **webhook** if you need instant reaction to Git commits.
11. **Prefer Kustomize overlays over per-environment branches** for promoting changes across dev → staging → production.
12. **Argo CD does not replace Jenkins.** You still need CI to test and build. Argo CD replaces the **CD** pipeline, and only **for Kubernetes**. **Flux CD** is the leading alternative implementing the same principles.
13. **Always check the docs for `apiVersion`.** The Application CRD's API version changes as the project moves from alpha toward stable.

[↑ Back to top](#table-of-contents)

---

*End of notes.*
