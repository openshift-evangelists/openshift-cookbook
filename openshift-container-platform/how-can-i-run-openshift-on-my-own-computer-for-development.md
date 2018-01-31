To quickly start up an OpenShift cluster locally inside of a virtual machine (VM), you can use [Minishift](https://www.openshift.org/minishift/). This isn’t an OpenShift distribution, but a tool which you can run to create a minimal VM which includes a container service. Minishift then downloads and launchs a pre-formatted container image containing OpenShift.

Running Minishift requires a hypervisor to run the VM containing OpenShift. Depending on your host operating system, you have the choice of the following hypervisors:

* macOS: xhyve (default), VirtualBox
* GNU/Linux: KVM (default), VirtualBox
* Windows: Hyper-V (default), VirtualBox

To download the latest Minishift release and view any release notes, visit the Minishift [releases](https://github.com/minishift/minishift/releases) page. Before using Minishift, ensure you check out the [installation instructions](https://docs.openshift.org/latest/minishift/getting-started/index.html) for any pre‐requisites your system must satisfy.

Minishift is based on the upstream OpenShift Origin project. If you want to use the exact same version of OpenShift as provided by Red Hat via a subscription to OpenShift Container Platform, you can sign up to the free [Red Hat Developers](https://developers.redhat.com) program and download the [Red Hat Container Development Kit](https://developers.redhat.com/products/cdk/overview/).
