So that other Container Runtimes aside from Docker can work with Kubernetes, the Open Container Initiative (OCI) was created. OCI is composed of imagespec and runtimespec.
- imagespec -> defines specifications on how an image should be built
- runtimespec -> standards for Container Runtime

Kubernetes needed to support Docker as it was still the most popular Container Runtime in the past so they created dockershim. This was necessary as Docker did not adhere to the OCI.

### containerd 
in version 1.24 Kubernetes discontinued dockershim. containerd is its own container runtime now. 
#### containerd tools
ctr - debugging tool
nerdctl - replaces the command "docker" for containerd

### crictl
Kubernetes tool
crictl - works the same as "nerdctl", it was created by the kubernetes community and works across all different container runtimes. Needs to be installed separately. Primarily a debugging tool.
Needs to be configured to the correct endpoint


