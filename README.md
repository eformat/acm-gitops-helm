# ACM GitOps - Helm Version

ACM, PolicyGenerator Kustomize Pugin, Helm

# 🤠 For the impatient

- Install RHACM Operator 2.6/2.7+ into your OpenShift4 cluster
- Label you local cluster with `useglobal=true` and `teamargo=true`
- Boostrap global ArgoCD for policy

```bash
oc apply -f bootstrap-acm-global-gitops/setup.yaml
```

- Generate manifests from [Helm using Kustomize](https://github.com/kubernetes-sigs/kustomize/blob/master/examples/chart.md)

```bash
kustomize build --enable-helm team-gitops-policy/helm-input > team-gitops-policy/generator-input/helm-generated.yaml
```

- Git add . ; Git commit - the `helm-generated.yaml` file.
- Install Team based ArgoCD's

```bash
oc apply -f applicationsets/team-argo-appset.yaml
```
