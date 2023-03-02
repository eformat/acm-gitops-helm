# ACM GitOps

[![ACM Team ArgoCD](http://img.youtube.com/vi/eGxPMkADAbc/0.jpg)](http://www.youtube.com/watch?v=eGxPMkADAbc "ACM Team ArgoCD")
## Quickly deploying ArgoCD ApplicationSets using RHACM's global-clusterset

A demonstration of deploying Team Based ArgoCD instances using RHACM's global `ClusterSet` and `PolicyGenerator`.

Out of the box there are several resources already deployed when you install ACM [you can read about them here](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.6/html-single/multicluster_engine/index#managedclustersets_global) in the documentation.

There is a namespace called `open-cluster-management-global-set` with a global object and binding.

```
oc get ManagedClusterSet global -o yaml
oc get ManagedClusterSetBinding global -n open-cluster-management-global-set -oyaml
```

## Bootstrap a Cluster Scoped ArgoCD for our Policies

We are going Bootstrap a cluster-scoped ArgoCD instance into the `open-cluster-management-global-set` namespace. We will deploy our Team ArgoCD's using ACM `Policy` that is generated using the `PolicyGenerator` [tool - the reference file is here](https://github.com/stolostron/policy-generator-plugin/blob/main/docs/policygenerator-reference.yaml).

```bash
oc apply -f bootstrap-acm-global-gitops/setup.yaml
```

This deploys:

* Subscription Resource

GitOps operator `Subscription`, including disabling the default ArgoCD and setting cluster-scoped connections for our namespaces - see `ARGOCD_CLUSTER_CONFIG_NAMESPACES` env.var that is part of the `Subscription` object. If your namespace is not added here, you will get namespace scoped connections for your ArgoCD, rather than all namespaces.

* GitOpsCluster Resource

This resource provides a Connection between ArgoCD-Server and the Placement (where to deploy exactly the Application).

* Placement Resource

We use a `Placement` resource for this global ArgoCD which deploys to a fleet of Clusters, where the Clusters needs to be labeled with `useglobal=true`.

* ArgoCD Resource

The CR for out global Argocd where we will deploy Policy. We configure ArgoCD to download the `PolicyGenerator` binary, and configure kustomize to run with the setting:

```yaml
kustomizeBuildOptions: --enable-alpha-plugins
```

## Deploy Team Based ArgoCD using Generated Policy

We are going to deploy ArgoCD for two teams now using the ACM `PolicyGenerator`.

The `PolicyGenerator` runs using kustomize. We specify the `generator-input/` folder - that holds our YAML manifests for each ArgoCD - in this case one for `fteam`, one for `zteam`.


You can run the `PolicyGenerator` from the CLI to test it out before deploying - download it using the [instructions here](https://github.com/stolostron/policy-generator-plugin/blob/main/README.md) e.g.

```bash
kustomize build --enable-alpha-plugins team-gitops-policy/
```

We specify the placement rule `placement-team-argo` - where the Clusters needs to be labeled with `teamargo=true`. We add some default compliance and control labels for grouping purposes in ACM Governance. We also set the `pruneObjectBehavior: "DeleteAll` so that if we delete the `ApplicationSet` the generated `Policy` s deleted and all objects are removed. For this to work, we must also set the `remediationAction` to `enforce` for our Policies.

One last configuration is to set the ArgoCD `IgnoreExtraneous` compare option - as Policy is generated we do not want ArgoCD to be our of sync for these objects.

```yaml
---
apiVersion: policy.open-cluster-management.io/v1
kind: PolicyGenerator
metadata:
  name: argocd-teams
placementBindingDefaults:
  name: argocd-teams
policyDefaults:
  placement:
    placementName: placement-team-argo
  categories:
    - CM Configuration Management
  complianceType: "musthave"
  controls: 
    - CM-2 Baseline Configuration
  consolidateManifests: false
  disabled: false
  namespace: open-cluster-management-global-set
  pruneObjectBehavior: "DeleteAll"
  remediationAction: enforce
  severity: medium
  standards:
    - generic
  policyAnnotations: {"argocd.argoproj.io/compare-options": "IgnoreExtraneous"}
policies:
  - name: team-gitops
    manifests:
      - path: generator-input/
```

To create our Team ArgoCD's run:

```bash
oc apply -f applicationsets/team-argo-appset.yaml
```

To delete them, remove the `AppSet` or remove the `teamargo=true` label from the cluster.

```bash
oc delete appset team-argo
```

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
