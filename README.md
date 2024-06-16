# GitOps Example Apps

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

## Kustomize and ApplicationSet

Use Kustomize and ArgoCD ApplicationSet to deploy an appliction to multi-envs.

On IaC repo: https://github.com/xdevops-caj-lab-gitops-cd/gitops-example-iac

Run below commands to create namespace (simulate for env) and ArgoCD applicationset.

```bash
oc apply -f apps/bgd
```

Open the route url of `bgd-dev` and `bgd-sit` to veify the color of dot is different.

Verify ArgoCD applications and OpenShift resources on ArgoCD Web UI.



## AppProject

We use `appproj-bgd` to organize ArgoCD applications.

### RBAC in AppProject

We set users under `bgd-group` has readonly permission for `approj-bdg` approject.

```yaml
  - name: read-only
    policies:
    - p, proj:appproj-bgd:read-only, applications, get, appproj-bgd/*, allow
    groups:
    - bgd-group
```

### Create a Group and Add a User into the group

Login OpenShift with `user1`.

Login ArgoCD Web UI with `user1`, but the user can't see any applications.

Add a group `bgd-group`, and add the user `user1` into this group:

Run below command with cluter admin user:
```bash
oc adm groups new bgd-group
oc adm groups add-users bgd-group user1
```

Verify the new created group:
```bash
oc get group bgd-group
```

Re-login ArgoCD Web UI with `user1`, the user can view the applciations under `appproj-bgd`, but can't operate the applications due to permission denied.



## App of Apps

TODO

## Syn Window

TODO


## Helm Chart

TODO

## Deploy to another cluster

TODO

## References

- https://github.com/redhat-developer/gitops-operator/
- https://github.com/redhat-developer/gitops-operator/blob/master/docs/OpenShift%20GitOps%20Usage%20Guide.md
- https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/
- https://argo-cd.readthedocs.io/en/latest/user-guide
