---
layout: post
category:
tags:
tagline:
---

I've had the luxury of trying out Docker and Kubernetes at work, and I thought: "would this work on a low(er) powered computer? When you check the requirements for many applications, running Docker itself is out of reach. As an example, my personal computer "only" has 8gb of ram, running the Docker daemon itself and a container ontop of it renders the computer near unusable (and sometimes has crashed the computer itself...)

Podman however, allows for it to run daemon-less. This is great, because the majority of time you're just interacting with containers via CLI. Furthermore the commands in general can just be switched over seamlessly between podman and docker.

What's next in this space for me?

One of the areas I'm actively thinking more about is how to ease the transition to Kubernetes. When I used Docker, I was exposed to `docker-compose` which has Kompose.io. However I found that Kompose.io only met ~80% of the needs when moving to Kubernetes, which isn't as "hands-off" as I wanted, especially when moving between local to cloud environments.

Next steps in my mind are reviewing:

- https://developers.redhat.com/blog/2019/01/29/podman-kubernetes-yaml
- https://www.redhat.com/sysadmin/podman-inside-kubernetes
- https://www.redhat.com/sysadmin/compose-kubernetes-podman
