# Jenkins Kubernetes RBAC Setup

Run these configurations on your Kubernetes master node to create a ServiceAccount, Role, RoleBinding, and Token Secret for Jenkins in the `webapps` namespace.

## Prerequisites
Ensure the `webapps` namespace exists before applying these files:
```bash
kubectl create namespace webapps
```

## 1. Create ServiceAccount (`service-account.yml`)
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

## 2. Create Role (`role.yml`)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - networking.k8s.io # Replaced deprecated 'extensions'
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - secrets
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingresses
      - jobs
      - limitranges
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

## 3. Bind Role to ServiceAccount (`bind.yml`)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
```

## 4. Generate ServiceAccount Token (`secret.yml`)
*Note: Added `namespace: webapps` so it links properly to the Jenkins ServiceAccount.*
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: jenkins-secret
  namespace: webapps
  annotations:
    kubernetes.io/service-account.name: jenkins
```

## Apply Configurations
```bash
kubectl apply -f service-account.yml
kubectl apply -f role.yml
kubectl apply -f bind.yml
kubectl apply -f secret.yml
```

