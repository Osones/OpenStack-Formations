# KUBERNETES : Installation

### Kubernetes : Minikube

- Outil permettant de démarrer rapidement un cluster mono-node Kubernetes localement
- Execute Kubernetes dans une machine virtuelle
- Nécessite des outils de virtualisation (VirtualBox, VMware Fusion, KVM, etc...)
- Supporte plusieurs systèmes d'exploitation : Linux, Mac OS, Windows
- Installation : <https://github.com/kubernetes/minikube#Installation>


### Kubernetes : Minikube

```console
$minikube start --kubernetes-version="v1.19.7"
Starting local Kubernetes v1.16.1 cluster...
Starting VM...
Getting VM IP address...
[...]
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.
```

### Kubernetes : Minikube

- Effectuer un diagnostic basique du cluster

```console
$ kubectl version
Client Version: v1.19.7
Server Version: v1.19.7
```


### Kubernetes : Minikube

```console
$ kubectl get componentstatuses
NAME                      STATUS    MESSAGE              ERROR
controller-manager         Healthy      ok
scheduler                  Healthy      ok
etcd-0                     Healthy      {"health": "true"}
```

### Installation de Kubernetes


- De nombreuses ressources présentes pour le déploiement de Kubernetes dans un environnement de production

- Un des outils est [kubeadm](https://github.com/kubernetes/kubeadm) utilisé pour rapidement démarrer un cluster Kubernetes

### Installation de Kubernetes avec Kubeadm

- Certains pré-requis sont nécessaires avant d'installer Kubernetes :
    - Désactiver le swap
    - Assurer que les ports requis soient ouverts : <https://kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports>
    - Installer une Container Runtime compatible CRI (containerd, CRI-O, Docker)

### Kubeadm

- Installer les composants Kubernetes (kubeadm, kubectl, kubelet) : <https://kubernetes.io/docs/setup/independent/install-kubeadm/>
- Exécuter `kubeadm init` sur le noeud master
- Exécuter `kubeadm join` sur les autres noeuds (avec le token fournit par la commande `kubeadm init`)
- Copier le fichier de configuration généré par `kubeadm init`
- Installer le plugin Réseau

### Kubeadm

En plus de l'installation de Kubernetes, Kubeadm peut :

- Renouveler les certificats du Control Plane
- Générer des certificats utilisateurs signés par Kubernetes
- Effectuer des upgrades de Kubernetes (`kubeadm upgrade`)

### Kubernetes managés "as a Service"

- Il existe des solutions managées pour Kubernetes sur les cloud publics :
    - AWS Elastic Kubernetes Services (EKS): <https://aws.amazon.com/eks/>
    - Azure Kubernetes Service (AKS): <https://azure.microsoft.com/en-us/services/kubernetes-service/>
    - Docker Universal Control Plane : <https://docs.docker.com/ee/ucp/>
    - Google Kubernetes Engine : <https://cloud.google.com/kubernetes-engine/>
    - Scaleway Kapsule : <https://www.scaleway.com/fr/kubernetes-kapsule/>
    - Alibaba Container Service for Kubernetes (ACK) <https://www.alibabacloud.com/fr/product/kubernetes>

### Installation de Kubernetes

- Via Ansible : kubespray <https://github.com/kubernetes-sigs/kubespray>
- Via Terraform : <https://github.com/poseidon/typhoon>
- Il existe d'autres projets open source basés sur le langage Go :
    - kube-aws : <https://github.com/kubernetes-incubator/kube-aws>
    - kops : <https://github.com/kubernetes/kops>

### Conformité kubernetes

Voici quelques outils permettant de certifier les déploiements des cluster kubernetes en terme de sécurité et de respects des standard

- Sonobuoy 
    - <https://github.com/vmware-tanzu/sonobuoy>
- Popeye
    - <https://github.com/derailed/popeye>
- kube-score
    - <https://github.com/zegl/kube-score>


