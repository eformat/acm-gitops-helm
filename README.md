# ACM GitOps - Helm Version

ACM, PolicyGenerator Kustomize Pugin, Helm

# 🤠 For the impatient

- Install RHACM Operator 2.6/2.7+ into your OpenShift4 cluster
- Label you local cluster with `useglobal=true` and `teamargo=true`
- Boostrap global ArgoCD for policy

```bash
oc apply -f bootstrap-acm-global-gitops/setup.yaml
```

- Install Team based ArgoCD's

```bash
oc apply -f applicationsets/team-argo-appset.yaml
```

# WIP

 This uses the [PR to implement kustomizeOptions](https://github.com/open-cluster-management-io/policy-generator-plugin/pull/109)

```yaml
policies:
  - name: team-gitops
    manifests:
      - path: helm-input/
        kustomizeOptions:
          enableAlphaPlugins: true
          enableHelm: true
```

Old init - put this back once PR merges:

```yaml
    initContainers:
    - name: policy-generator-install
      image: registry.redhat.io/rhacm2/multicluster-operators-subscription-rhel8:v2.7.0-57
      command: ["/bin/bash"]
      args: ["-c", "cp /etc/kustomize/plugin/policy.open-cluster-management.io/v1/policygenerator/PolicyGenerator /policy-generator/PolicyGenerator"]
      volumeMounts:
      - mountPath: /policy-generator
        name: policy-generator
```