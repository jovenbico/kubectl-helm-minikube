I'm using this setup as my playground

## Setup the environment

### Using Multi-Node Clusters
```
$ minikube start \
 --kubernetes-version 1.24.7 \
 --container-runtime=containerd \
 --nodes 3 \
 --cni calico
```
[Using Multi-Node Clusters](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/)

[Enabling Calico on a minikube cluster](https://minikube.sigs.k8s.io/docs/handbook/network_policy/#enabling-calico-on-a-minikube-cluster)

[Runtime Configuration](https://minikube.sigs.k8s.io/docs/handbook/config/#runtime-configuration)
