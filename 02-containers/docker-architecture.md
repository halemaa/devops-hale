\# Docker Architecture



\## What it is

Docker is a platform for packaging applications and their dependencies into

lightweight, portable containers that share the host OS kernel.



\## The mental model

```mermaid

flowchart LR

&#x20;   CLI\[Docker CLI] --> Daemon\[dockerd]

&#x20;   Daemon --> Containerd\[containerd]

&#x20;   Containerd --> Runc\[runc]

&#x20;   Runc --> Container\[Running Container]

&#x20;   Daemon --> Registry\[(Registry: Docker Hub / GHCR)]

```



\## Key components

\- \*\*Docker CLI\*\* — the `docker` command you type.

\- \*\*dockerd (daemon)\*\* — receives CLI commands, manages images, containers, networks, volumes.

\- \*\*containerd\*\* — the container runtime that pulls images and manages lifecycle.

\- \*\*runc\*\* — the low-level tool that actually spawns the container using Linux namespaces and cgroups.

\- \*\*Image\*\* — a read-only template built from a Dockerfile.

\- \*\*Container\*\* — a running instance of an image.

\- \*\*Registry\*\* — where images live (Docker Hub, GitHub Container Registry, ECR).



\## How a container is isolated

\- \*\*Namespaces\*\* — isolate what the container can \*see\* (PID, network, mount, user).

\- \*\*cgroups\*\* — isolate what the container can \*use\* (CPU, memory, I/O).

\- Containers share the host kernel — this is why they're lighter than VMs.



\## Common gotchas

\- Containers are ephemeral. Anything not written to a volume disappears on removal.

\- `latest` is not a version — it's a moving tag. Pin your image tags.

\- The `root` user inside a container is `root` on the host if not remapped.



\## What I want to try

Build a small image from a Dockerfile, push it to GitHub Container Registry,

and inspect the layers with `docker history`.



\## Sources

\- Docker official docs

\- "Docker Deep Dive" — Nigel Poulton

