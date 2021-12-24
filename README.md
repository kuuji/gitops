# Argocd


## gitops repo structure


```
clusters
    {cluster_name}
        system 
        ops
        game
        etc.
shared
    system
        istio
        cert-manager
    ops
        nginx
            nginx-header_helm.yaml
    game
    etc.
```

`clusters/{cluster_name}/system` contains the system components that are not shared. For example argocd only exists on the argocd cluster
gha-controller only exists on the ci cluster
etc.



`shared/system/` contains applicationSets, those resources generate Application for every cluster according to some parameters (cluster/git files/list/more complex if we want to). It doesn't have to be on all clusters, but if it's more than 1 it's useful to have an applicationSet for it. 
These are the things we always install and have very little variation between clusters. For example istio, cert-manager, etc.
We should aim to have as much as possible here.

`shared/ops/nginx/nginx_helm.yaml` is an applicationSet that takes a remote helm chart and installs it in a list of cluster. values can be templated directly in the applicationSet. This design while being the most straightforward does not allow to permanently override values for a given cluster. It's good for things that have very little variation between clusters.


`shared/ops/nginx/nginx_helm_umbrella.yaml` is an applicationSet that looks for a helm chart in `clusters/{cluster_name}/ops/nginx`
This folders contains:
- A Chart.yaml file with a dependency to a chart that installs nginx-header
- A values.yaml file that has values specific to the cluster


terraform
- create cluster
- create cluster dependencies
    - acm
    - iam role
- install argo via helm provider
- install alb ingress
- bootstrap argo
  - create an argo application that looks at the clusters/argo folder in gitops
  - clusters/argo folder contains bootstrap.yaml <- from now on we manage argo via argo
  - clusters/argo/system/argocd contains all argocd required config
  - clusters/argo/root.yaml creates an application that looks at shared/
  - shared/system/istio contains everything to bootstrap istio on any cluster