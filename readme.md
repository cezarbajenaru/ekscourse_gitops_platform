~/projects/eks_course_platform_gitops
  ├── argo/
  │   ├── bootstrap.yaml
  │   └── install/
  │      └── kustomization.yaml
         ├── helm-argocd.yaml
         └── values.yaml

  │   └── applications/
  │       └── <App-of-Apps>
  └── apps/
      └── podinfo/
          ├── deployment.yaml
          └── service.yaml
################# come back and recreate the whole tree everytime you add stuff like folders and files. Helps readability#####

step 0 - have the infrastructure done with Terraform

step 1 - apply bootstrap.yaml to install argo and link it to the cluster to the repo
step 2 - After boostrap it automatically Upgrades the simple ArgoCD install created at at step 1  |  install/ will use it's own resources to configure itself TLS, RBAC, Autosync rules, server service type, Ingress, LoadBalancer(our case), Image updates, high availability configurations - ensures no drift, reproducability, version controlled platform config

step 3 - # username is admin and you get the generated pass though this command
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo

Step 4 access you apps via adresses like IP:port


In short, Argo applies only what is defined in argo/applications:
   Under argo/applications/ folder , apps/yaml file contains YAML code for ArgoCD to execute the apps that reside in main apps/aplication_folder_name/k8s_manifests.yaml (deployment, service etc)
In apps.yaml (the orchestrator) you can have a separate different repo for each app. In this file, YAMLs from different apps are separated with --- ( this does not break the YAML)

```
bootstrap.yaml
   ↓ (apply manually once)
ArgoCD minimal
   ↓ (reads from Git)
argo/applications/*   ← App-of-apps definitions
   ↓
argo/install/*        ← Full ArgoCD installation (self-managed)
apps/<app-name>/*     ← Application manifests (podinfo, etc.)

```


There are two common ways to manage ArgoCD installation in GitOps:
- Raw YAML manifests - not recomended - hard to upgade, no values overrides
- Helm Charts - THE WAY TO GO - configuration lives in values.yaml
```
argo/
└── install/
    ├── kustomization.yaml
    ├── helm-argocd.yaml  # defines a Helm release
    └── values.yaml

helm-argocd.yaml lets argo upgrade itself anytime with Helm chart updates

bootstrap.yaml
   ↓ (apply manually once)
ArgoCD minimal
   ↓ (reads from Git)
argo/applications/*   ← App-of-apps definitions
   ↓
argo/install/*        ← Full ArgoCD installation (self-managed)
apps/<app-name>/*     ← Application manifests (podinfo, etc.)
```

The best part of using Argo is that each team can use it's own repo and each time they commit something to that particular repo, Argo deploys it. It can be a dev cluster or even production. This is practically autonomous microservice deployments with a centralized platform that you can govern with allmost a single app like Argo. Argo uses git differences and deploys, does health checks and does canary or rollbacks

THere are no passwords, no kubectl for me

Discover Argo for multimple environments

dev cluster can be made fully automatic
staging cluster can be semi-automatic with approve in Argo UI manually
production cluster can be PR based + gating rules
Practically git each commit auto-transfers to real infrastructure and real users(production case)




