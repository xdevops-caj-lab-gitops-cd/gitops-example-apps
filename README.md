# GitOps Example Apps


## Check list

- Test deploy a simple application to different namespace
  - on ArgoCD Web UI
  - on command

## Create a Group and Add a User into the group

The RBAC of default `openshift-gitops` ArgoCD instance:

```yaml
  rbac:
    defaultPolicy: ''
    policy: |
      g, system:cluster-admins, role:admin
      g, cluster-admins, role:admin
    scopes: '[groups]'
```

Notes: Only users under `system:cluster-admins` and `cluster-admins` groups have `role:admin`, other users only have `role:readonly` permissions.

Check existing groups:
```bash
oc get groups
```

Add a group `cluster-admins`, and add a user `admin` into this group:
```bash
oc adm groups new cluster-admins
oc adm groups add-users cluster-admins admin
```

Check groups and users:
```bash
oc get groups
```

## Deploy resources to a different namespace

To grant Argo CD the permissions to manage resources in multiple namespaces, we need to configure the namespace with a label "Argo CD.argoproj.io/managed-by" and the value being the namespace of the Argo CD instance meant to manage the namespace.

```bash
oc label namespace <namespace> argocd.argoproj.io/managed-by=<namespace of argocd instance>
```

## Simple App


### Spring Petclinic

#### Deploy on ArgoCD Web UI

Need create namespace firstly, and let ArgoCD instance manage this namespace before application deployment.

```bash
oc create namespace spring-petclinic
oc label namespace spring-petclinic argocd.argoproj.io/managed-by=openshift-gitops
```

Application details:
- Application Name: `spring-petclinic`
- Project Name: `default`
- Sync Policy: `Automatic`
- Source:
  - Repository URL: `https://github.com/xdevops-caj-lab-gitops-cd/gitops-example-apps.git`
  - Target Revision: `HEAD`
  - Path: `spring-petclinic`
- Destination:
  - Cluster URL: `https://kubernetes.default.svc`
  - Namespace: `spring-petclinic`

#### Deploy by command

On IaC repo: https://github.com/xdevops-caj-lab-gitops-cd/gitops-example-iac

Run below commands to create namespace and ArgoCD application.

```bash
oc apply -f apps/spring-petclinic
```

#### Clean up

```bash
oc delete application spring-petclinic -n openshift-gitops
oc delete namespace  spring-petclinic
```

## References

- https://github.com/redhat-developer/gitops-operator/
- https://github.com/redhat-developer/gitops-operator/blob/master/docs/OpenShift%20GitOps%20Usage%20Guide.md
- https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/

