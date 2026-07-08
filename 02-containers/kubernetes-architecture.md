\# Kubernetes Architecture



\## What it is

Kubernetes (K8s) is an orchestrator: you tell it what you want running

("3 copies of my web app, always"), and it figures out how to make that true

across a cluster of machines. It handles scheduling, self-healing, scaling,

networking, and rollouts.



\## The mental model

```mermaid

flowchart TB

&#x20;   subgraph ControlPlane\["Control Plane (the brain)"]

&#x20;       API\[API Server]

&#x20;       etcd\[(etcd)]

&#x20;       Scheduler\[Scheduler]

&#x20;       CM\[Controller Manager]

&#x20;       API <--> etcd

&#x20;       API <--> Scheduler

&#x20;       API <--> CM

&#x20;   end

&#x20;   subgraph Node1\["Worker Node 1"]

&#x20;       Kubelet1\[Kubelet]

&#x20;       Proxy1\[kube-proxy]

&#x20;       Runtime1\[Container Runtime]

&#x20;       Pod1\[Pod]

&#x20;       Pod2\[Pod]

&#x20;       Kubelet1 --> Runtime1

&#x20;       Runtime1 --> Pod1

&#x20;       Runtime1 --> Pod2

&#x20;   end

&#x20;   subgraph Node2\["Worker Node 2"]

&#x20;       Kubelet2\[Kubelet]

&#x20;       Proxy2\[kube-proxy]

&#x20;       Runtime2\[Container Runtime]

&#x20;       Pod3\[Pod]

&#x20;       Kubelet2 --> Runtime2

&#x20;       Runtime2 --> Pod3

&#x20;   end

&#x20;   User\[kubectl / user] --> API

&#x20;   API --> Kubelet1

&#x20;   API --> Kubelet2

```



\## Control Plane components

\- \*\*API Server (`kube-apiserver`)\*\* — the front door. Every action (from kubectl, controllers, kubelets) goes through here.

\- \*\*etcd\*\* — the source of truth. Key-value store holding the entire cluster state. Back it up.

\- \*\*Scheduler (`kube-scheduler`)\*\* — decides which node a new pod runs on, based on resources, taints, affinity rules.

\- \*\*Controller Manager (`kube-controller-manager`)\*\* — runs the control loops that make reality match desired state (Deployment controller, ReplicaSet controller, Node controller, etc.).

\- \*\*Cloud Controller Manager\*\* — integrates with the cloud provider (load balancers, volumes, nodes).



\## Worker Node components

\- \*\*Kubelet\*\* — the agent on every node. Talks to the API server, makes sure the pods it's told to run are actually running.

\- \*\*kube-proxy\*\* — handles network rules for Services (iptables / IPVS).

\- \*\*Container Runtime\*\* — actually runs containers. `containerd` or CRI-O today (Docker is deprecated as a runtime).



\## Core objects (the vocabulary)

\- \*\*Pod\*\* — the smallest unit. One or more tightly coupled containers sharing network + storage. Ephemeral.

\- \*\*ReplicaSet\*\* — keeps N copies of a Pod running.

\- \*\*Deployment\*\* — manages ReplicaSets, handles rolling updates and rollbacks. You almost never create ReplicaSets directly.

\- \*\*Service\*\* — stable network endpoint for a set of Pods. Types: ClusterIP (internal), NodePort (external via node), LoadBalancer (external via cloud LB).

\- \*\*Ingress\*\* — HTTP(S) routing to Services. Needs an Ingress Controller (NGINX, Traefik) to actually work.

\- \*\*ConfigMap\*\* — non-secret config.

\- \*\*Secret\*\* — sensitive config (base64-encoded, not encrypted by default — turn on encryption at rest).

\- \*\*Namespace\*\* — logical partitioning of the cluster.

\- \*\*PersistentVolume / PersistentVolumeClaim\*\* — storage abstraction.

\- \*\*StatefulSet\*\* — for stateful apps (databases). Stable network IDs, ordered rollout.

\- \*\*DaemonSet\*\* — one pod per node (log agents, monitoring).

\- \*\*Job / CronJob\*\* — run-to-completion and scheduled tasks.



\## Declarative

