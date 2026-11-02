---
name: container-k8s-escape
description: Container and Kubernetes escape playbook for authorized environments — identifying container context, Docker misconfigurations (socket exposure, privileged containers, hostPath, capabilities), Kubernetes API and kubelet abuse, service-account token pivots, RBAC enumeration, and pod-to-host escape. Use whenever containers, a K8s API port, or a foothold lands inside a container or on a container host.
metadata:
  short-description: Container & K8s escape playbook
  category: cloud
---

# Container / Kubernetes Escape

> Authorized engagements only (see `authorized-use`). The recurring real-world path: exposed K8s
> API or kubelet → exec in a privileged pod → host root — plus classic Docker misconfigurations on
> single hosts.

## 1. Know exactly where you are
- In a container? Check `/proc/1/cgroup`, mount table, hostname patterns, `capsh`, and the runtime
  (`docker.sock`, containerd sockets). On the host? Different playbook (see OS privesc skills).
- Note the node/cluster context: a pod on a managed node is one step from cluster-level identity.

## 2. Docker-style escapes (single host)
- **Exposed docker.sock**: mount or launch privileged containers via the socket — instant host
  root; check for it on filesystems and TCP.
- **Privileged containers**: host devices, cgroup release_agent, and nested runtime escapes.
- **Dangerous capabilities** (`CAP_SYS_ADMIN`, `CAP_SYS_PTRACE`, `CAP_DAC_READ_SEARCH`): map each to
  its documented escape.
- **hostPath volumes** and writable paths that reach host services (cron, systemd units).
- Kernel CVEs are the noisy last resort.

## 3. Kubernetes control plane
- **API server (6443/8443/844x)**: test anonymous access first, then any token you hold. List pods,
  secrets, and permissions (`SelfSubjectRulesReview`-style "what can I do") before acting.
- **Service-account tokens** live in every pod
  (`/var/run/secrets/kubernetes.io/serviceaccount/`) — harvest and test them against the API; a
  single over-privileged token is cluster-level compromise.
- **RBAC enumeration**: role bindings, escalate verbs (`exec`, `create pods`, `create
  privileged`), secret reads — the shortest documented path wins.
- **exec into other pods** for their tokens/env/secrets, and to reach node-local services.

## 4. Kubelet and node services
- **Kubelet API (10250)**: anonymous/weak auth on `run`/`exec`/health endpoints is a direct node
  shell path; check `/pods` disclosure for pod inventory.
- etcd, scheduler, and controller-manager ports — enumerate exposure; etcd read is cluster memory.

## 5. Pivot to cloud/host
- Image pull secrets, cloud instance roles, and registry credentials in pod specs/env are the
  cloud bridge (see `cloud-attack-paths`); node compromise is host compromise — stop and document
  once proven.

## 6. Operate cleanly
- K8s clusters audit-log aggressively: every API call is logged. Minimize object reads, avoid
  destructive verbs, record namespace/SA for every action, and clean up any pods you created.

---
*Patterns distilled clean-room from public community lab writeup material (Kubernetes and Docker
attack paths are extensively exercised in public labs). No writeup text or flags reproduced.*
