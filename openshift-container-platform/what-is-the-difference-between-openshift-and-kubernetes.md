[Kubernetes](https://kubernetes.io/) implements a system for automating deployment, scaling and management of containerized applications. It is often referred to as being a Container as a Service (CaaS).

Kubernetes alone does not provide any support for building the container image it runs. You need to run a build tool to create your application image on a separate system and push that to an image registry from which it can be deployed. This is because CaaS focuses on just running containers.

OpenShift builds on top of, and bundles, Kubernetes, to implement a Platform as Service (PaaS) environment which is more friendly to developers, as well as provide the additional tools and services needed by operations to implement a comprehensive container application platform.
