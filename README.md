# ACM GitOps - Helm Version

ACM, PolicyGenerator Kustomize Pugin, Helm

# 🤠 For the impatient

- Install RHACM Operator into your OpenShift4 cluster
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

[Add POLICY_GEN_ENABLE_HELM env to generator config](https://github.com/open-cluster-management-io/policy-collection/pull/443)


Init - put this back once PR merges in upstream image, for now carrying [quay.io/eformat/policy-generator-plugin-image:v1.13.0](https://quay.io/repository/eformat/policy-generator-plugin-image)

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