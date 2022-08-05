# air gap bootstrap

A bootstrap to a user from scratch in air-gap to a highly available kubernetes cluster.

## Why?
Solving one of the chicken and egg scenarios in Air Gapped environments. When standing up infrastructure, DNS is required for kubernetes and any FQDN routing that occurs. Load balancing is required for highly available clusters to allow failover.

This would allow the DNS and loadbalancing dependency to be embedded within the kubernetes cluster. As each node is deployed, it contains an externally exposed DNS server and tcp loadbalancer for purpose of resolving both itself and other kubernetes nodes by FQDN. Highly available clusters rely on DNS resolution for joining nodes to the cluster.

## Air Gapped How-To
- Deploy first 'server' node
    - Wait for initialization and validate health
- Deploy helm chart for bootstrap (daemonset)
    - DNS
        - ConfigMap is created for expected configuration
            - DNS record for each node (k8s-server-01.domain, k8s-agent-01.domain)
            - A cluster record - which points to the node itself (this could be tricky...)
                - Allows for failover, no hardcoded address to resolve, instead each node can direct required traffic to [localLB](https://github.com/brandtkeller/localLB)
                - initContainer and emptyDir?
                    - Loses the auto-reload value, but this may be null with GitOps anyway
                - (k8s-cluster.domain)
                - Can we run a quick sed with output to target location?
                    - Would allow the use of environment variable
                    - [hostIP](https://stackoverflow.com/questions/60794419/kubernetes-pod-yaml-set-as-env-var-the-pod-ip-and-port)
        - First pod should deploy and serve traffic
    - Load Balancer
        - ConfigMap is created for expected configuration
            - This may have to be static in nature - meaning that the full configuration will need to be deployed from the start
- Deploy 1 -> N additional 'server' nodes
    - Each spin-up a externally-exposed dns pod


## Other Uses
- Highly available DNS service
    - Can be configured to provide DNS for entire environment to include downstream systems
- Resolving web traffic to expected hosts