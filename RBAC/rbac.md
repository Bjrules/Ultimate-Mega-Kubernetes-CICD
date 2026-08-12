# RBAC
 
 RBAC YAML configuration for the `jenkins` ServiceAccount, Role, RoleBinding, ClusterRole, and ClusterRoleBinding to ensure the ServiceAccount can create all the resources in your YAML file, including dynamic provisioning with StorageClasses and PersistentVolumes.

### **1. ServiceAccount**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```


### **2. Role**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-role
  namespace: webapps
rules:
  # Permissions for core API resources
  - apiGroups: [""]
    resources:
      - secrets
      - configmaps
      - persistentvolumeclaims
      - services
      - pods
    verbs: ["get", "list", "watch", "create", "update", "delete","patch"]

  # Permissions for apps API group
  - apiGroups: ["apps"]
    resources:
      - deployments
      - replicasets
    verbs: ["get", "list", "watch", "create", "update", "delete","patch"]

  # Permissions for networking API group
  - apiGroups: ["networking.k8s.io"]
    resources:
      - ingresses
    verbs: ["get", "list", "watch", "create", "update", "delete","patch"]

  # Permissions for autoscaling API group
  - apiGroups: ["autoscaling"]
    resources:
      - horizontalpodautoscalers
    verbs: ["get", "list", "watch", "create", "update", "delete","patch"]
```


### **3. RoleBinding**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
```


### **4. ClusterRole**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-cluster-role
rules:
  - apiGroups: [""]
    resources:
      - persistentvolumes
    verbs: ["get", "list", "watch", "create", "update", "delete"]
  - apiGroups: ["storage.k8s.io"]
    resources:
      - storageclasses
    verbs: ["get", "list", "watch", "create", "update", "delete"]
```


### **5. ClusterRoleBinding**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-cluster-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-cluster-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
```



### **Explanation of Permissions**

1. **ServiceAccount**:  
   - The `jenkins` ServiceAccount is created in the `webapps` namespace.

2. **Role**:
   - Grants access to namespace-specific resources:
     - **Secrets**, **ConfigMaps**, **PersistentVolumeClaims**, **Services**, and **Pods**.
     - **Deployments** and **ReplicaSets** under the `apps` API group.

3. **RoleBinding**:
   - Binds the `jenkins` Role to the ServiceAccount in the `webapps` namespace.

4. **ClusterRole**:
   - Grants access to cluster-wide resources:
     - **PersistentVolumes** (required for dynamic provisioning).
     - **StorageClasses** (required to create and manage storage classes for dynamic PV provisioning).

5. **ClusterRoleBinding**:
   - Binds the `jenkins` ClusterRole to the ServiceAccount for cluster-wide operations.



### **How to Apply the YAML Files**
1. Save each YAML snippet as a separate file:
   - `serviceaccount.yaml`
   - `role.yaml`
   - `rolebinding.yaml`
   - `clusterrole.yaml`
   - `clusterrolebinding.yaml`

2. Apply them in the following order:
> Note that the namespace were they are to be deployed (webapps) is already defined  in the yaml file during creation so no need to specify it during kubectl apply. 
   ```bash
   kubectl apply -f serviceaccount.yaml
   kubectl apply -f role.yaml
   kubectl apply -f rolebinding.yaml
   kubectl apply -f clusterrole.yaml
   kubectl apply -f clusterrolebinding.yaml
   ```

3. Verify the ServiceAccount has the expected permissions:
   ```bash
   kubectl auth can-i create secrets --as=system:serviceaccount:webapps:jenkins -n webapps
   kubectl auth can-i create storageclasses --as=system:serviceaccount:webapps:jenkins
   kubectl auth can-i create persistentvolumes --as=system:serviceaccount:webapps:jenkins
   ```

### Generate token using service account in the namespace
[Create Token](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#:~:text=To%20create%20a%20non%2Dexpiring,with%20that%20generated%20token%20data.)

```
kubectl apply -f secret-file.yaml -n webapps
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sa-secret
  annotations:
    kubernetes.io/service-account.name: "jenkins"
type: kubernetes.io/service-account-token

```
#### kindly get the token using this command
```
kubectl describe secret sa-secret -n webapps

```
#### copy the token as it will be used for Jenkins-Kubernetes Authentication later.kindly refer to the jenkinsfile of the Ultimate-Mega-Kubernetes-CD Repo
```
[Kubernetes Secret for jenkins service account]
eyJhbGciOiJSUzI1NiIsImtpZCI6IjRYQzk0TVd2X21PTkhHeDVVMWE3Um5JbklCLUMwODNWaHFRMzVoeHhYcDQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InNhLXNlY3JldCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiOWVhMGMxNjktNTIyYS00NTNlLThmZmUtMjRiZGIzYWI1MDAwIiwic3Voic3lzdGVtOnNlcnZpY2VhY2NvdW50OndlYmFwcHM6amVua2lucyJ9.a-CKgaUbmsPKb0mcK657kA7UkP8In7FdXoa-GfAksDfhc5ZUoAq-Eg-ckhkLQYG68Qw6TNfvFUiWgZ64B8xARzia1kAKDCDfXlZEv69GLc_MU5C7gtriuKePMxiJePCqUFQTYAFOb8X4d59bjEfsa5fNOcZIjAvmgj7VAcFyREP-n4nOdXgFhm6ulFGXA9v7JYmsVH2WFApH-NjXK8WNBKeS0Gi09nacw7xh9UxwiF2-IUVeXXrDxoegF5N5-uLLRP0DjQsmTPfRgqpnyih7yBdTp629t4EuavU3E1TG5Xsf44SvBTddk3Q613asQBRBSe-xgGVrmtmhhElGov4DwQ

```
> See ScreenShots
