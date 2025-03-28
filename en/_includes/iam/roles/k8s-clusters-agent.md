### {{ roles.k8s.clusters.agent }} {#k8s-clusters-agent}

`{{ roles.k8s.clusters.agent }}`: Special role for the [{{ k8s }} cluster](../../../managed-kubernetes/concepts/index.md#kubernetes-cluster)[service account](../../../iam/concepts/users/service-accounts.md). It enables you to create [node groups](../../../managed-kubernetes/concepts/index.md#node-group), disks, and internal load balancers. You can use previously created [{{ kms-full-name }} keys](../../../kms/concepts/key.md) to encrypt and decrypt secrets and connect previously created [security groups](../../../managed-kubernetes/operations/connect/security-groups.md). When combined with the `load-balancer.admin` role, it enables you to create a network load balancer with a [public IP address](../../../vpc/concepts/address.md#public-addresses). It includes the following roles:
* `{{ roles.k8s.tunnelClusters.agent }}`
* `vpc.privateAdmin`
